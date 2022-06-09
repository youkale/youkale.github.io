# 线上服务器TCP连接相关

---

### 为什么优化点在网络？
随着云平台的兴起, 传统自建数据库或者其他业务中间件的情况会比较少，而我们系统也主要集中在调用平台提供的服务。

因此今天主要讲的是网络相关内容。

### TCP-4次挥手

```sequence
alice --> bob: FIN [1]
alice --> alice: FIN_WAIT_1 [2] 
bob -- alice: ACK [3]
alice -- alice: FIN_WAIT_2
bob --> bob: CLOSE_WAIT
bob --> alice: FIN ACK
bob --> bob: LAST_ACK
alice --> bob: ACK
alice -- alice: TIME_WAIT, 等待2MSL
bob --> bob: 关闭
alice --> alice: 2MSL关闭
```

```
alice -> bob: 分手吧。 (fin_wait_1)
bob -> alice: ok (close_wait)
alice: (fin_wait_2)
bob -> alice: 来呀，分手吧, (last_ack)
alice -> bob: ok ，(TIME-WAIT + 2MSL)
bob: CLOSE
```



### 为什么在排查TCP连接的时候要去检查TIME-WAIT? 

- 可用端口数量是有限的, `cat /proc/sys/net/ipv4/ip_local_port_range` 

 ```
    cat /proc/sys/net/ipv4/ip_local_port_range
    32768   60999
 ```

- 2MSL：表示段的最大生命周期，一般为2分钟(2 * 60s), 可以通过 `cat /proc/sys/net/ipv4/tcp_fin_timeout`进行查看

> 因此如果存在大量的timewait表示可用的连接减少，同时占用着资源，无法应对突发流量。

### 获取相关信息：

- 查看内核参数: ` sysctl -p |grep net`

  - ```shell
    [root@iz8vb3gm51vgkvb60twt46z ~]# sysctl -p |grep net
    net.ipv4.tcp_max_tw_buckets = 10000
    ...
    ```

- 统计TCP：

 ```shell
    [root@iz8vb3gm51vgkvb60twt46z ~]# ss -nat |awk '{x[$1] +=1}END{for(i in x) print i,x[i]}'
    LISTEN 13
    ESTAB 333
    CLOSE-WAIT 281
    State 1
    TIME-WAIT 9530
 ```

- 产生服务器：`ss -nat | awk '/TIME-WAIT/ {print $5}' | awk  -F ":" '{x[$1]+=1} END {for(i in x) print i, x[i]}'`

 ```shell
  [root@iz8vb3gm51vgkvb60twt46z ~]# ss -nat | awk '/TIME-WAIT/ {print $5}' | awk  -F ":" '{x[$1]+=1} END {for(i in x) print i, x[i]}'
  106.11.208.135 46
  14.119.64.132 9
  14.119.64.133 6
  14.215.62.21 4
  14.215.62.24 12
  101.89.47.18 4
  221.130.19.101 10
  203.119.169.6 148
  ....
 ```



通过查看内核参数`net.ipv4.tcp_max_tw_buckets` 得知当前服务器最大time-wait数量为: `10000`

通过`cat /proc/sys/net/ipv4/ip_local_port_range`实际可用端口数为: `61000 - 32768 = 28231`

### 解决方法

- 将time-wait留在客户端。（需要代码调整）

- 连接池化。(需要代码调整）

- 调整`tcp_max_tw_buckets` 在可用端口数范围内. (保证足够可用内存, 没有新的请求)

- 启用`net.ipv4.tcp_tw_reuse`, 该选项可用将进入到time-wait状态的连接重复使用。默认该选项是关闭的。
  > RFC1323 提出了一组TCP扩展，以改善高带宽路径的性能。除其他外，它定义了一个新的TCP选项，携带两个四字节的时间戳字段。第一个是发送该选项的TCP的时间戳时钟的当前值，而第二个则是从远程主机收到的最新时间戳。
  >
  > 通过启用net.ipv4.tcp_tw_reuse，如果新的时间戳严格大于前一个连接记录的最新时间戳，Linux将重新使用一个处于TIME-WAIT状态的现有连接，用于新的出站连接：一个处于TIME-WAIT状态的出站连接可以在短短一秒钟后被重新使用。 
  - 另外改选项生效需要配合 `net.ipv4.tcp_timestamps` 参数才能生效。
    - 正常情况下新连接的发送时间都大于之前一个连接的时间戳，不知道是否有其他情况。
    - 该参数是TCP的一个扩展，分别记录发送时间戳和从远程服务器收到时间戳
    - 在`tcp_tw_resuse`不生效的时候，可以看看远程服务器是否开启。

### 错误方法

- 打开 `net.ipv4.tcp_tw_recycle` 
	> 当远程主机是一个NAT设备时，关于时间戳的条件将禁止所有的主机在一分钟内连接，除了NAT设备后面的一台，因为它们不共享相同的时间戳时钟。如果有疑问，最好禁用这个选项，因为它导致了难以检测和难以诊断的问题。
	
	- 该参数在新版的内核中已经移除



## 线上服务错误的连接断开

### 问题：

> 在通过服务器调用远程某服务的时候，经常性出现`Read Timeout`. 
> Read timed out 是在调用socket read后，指定时间内未收到响应时 jdk抛出的异常， 比如: http response size = 10k, socket read = 4k, read timeout = 3s，那么每次读取限于3秒内读取完毕。简单理解就是在(e)poll的间隔时间.

- 通过`curl`对远程服务进行请求，发现请求正常，并未出现`Read Timeout`. 基本可以判断并非服务器内核参数问题.
- 通过tcpdump抓包: `tcpdump -i eth0 'port 80' -w 1.cap` 

> 从蓝色部分来看，客户端发出请求到服务器80端口后，并未收到响应，连接非正常关闭了。
> ![online-server-1](/images/online-server-1.png)
> 而这个从80端口返回的包，刚好上上一个连接错误断开后返回的。因为本地的连接已经销毁了，通过4元组已经找不到对应的连接，于是客户端-rst->服务80.
> ![online-server-2](/images/online-server-2.png)
> 这个图能直观看出之间的通信，左边80代表服务端口，右边代表客户端端口。
> ![online-server-3](/images/online-server-3.png)

### 解决方法: 

> 在使用 `WebUtil.doPostJson` 方法时候，使用默认不带`readTimeout`参数的方法的时候，会默认使用`60s`超时的读取超时时间。

- 使用带超时时间重载的方法，并设置超时时间为`0`

  ```java
  doPostJson(String url, String json, String charset, int connectTimeout, int readTimeout,
      Map<String, String> headerMap) throws IOException 
  ```



## 常见HttpClient优化方法

- Http连接管理 `PoolingHttpClientConnectionManager` 
```java
PoolingHttpClientConnectionManager connManager 
  = new PoolingHttpClientConnectionManager();
connManager.setMaxTotal(5);
connManager.setDefaultMaxPerRoute(4);
HttpHost host = new HttpHost("www.baidu.com", 80);
connManager.setMaxPerRoute(new HttpRoute(host), 5);
```
- 使用请求重试 `HttpRequestRetryHandler`
```java
 CloseableHttpClient httpclient = HttpClients.custom()
                .setRetryHandler(retryHandler())
                .build();

public HttpRequestRetryHandler  retryHandler() {
	...
}
```


## 参考

[RFC1323](https://datatracker.ietf.org/doc/html/rfc1323)
[浅谈Java中的TCP超时](https://hoswey.github.io/2019/07/23/%E6%B5%85%E8%B0%88Java%E4%B8%AD%E7%9A%84TCP%E8%B6%85%E6%97%B6/)
[HttpClient RetryHandler](https://www.jianshu.com/p/ade80fe11f58)
[Apache HttpClient Example](https://www.baeldung.com/httpclient-connection-management)