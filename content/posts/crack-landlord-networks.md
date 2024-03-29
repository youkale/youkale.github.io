---
title: 破解房东网络
date: 2015-11-21 03:18:43
tag: 网络
---

## 前言:
>最近搬到一个新地方,没有网,对于一个程序员没有网络是多么痛苦的一件事情呀.
还好房东留下了一个RJ-45的网络接口.哈哈哈哈,这不是给我机会了么?
好,立马插入网线来看看有没什么可以好玩的东西.

### 开始干活:

插入DHCP发现就获取到了IP

```bash
$ ifconfig
	eth0    Link encap:Ethernet  HWaddr C6:D6:06:33:xx:xx  
          inet addr:192.169.31.23  Bcast:192.169.30.255  Mask:255.255.255.0
          inet6 addr: fe80::c4d6:6ff:fe33:933b/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:25961173 errors:0 dropped:17 overruns:0 frame:0
          TX packets:18604131 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:27916381007 (25.9 GiB)  TX bytes:1554819614 (1.4 GiB)
 ```

看看路由信息

```bash
$ route -n
	Kernel IP routing table
	Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
	0.0.0.0         192.169.30.1    0.0.0.0         UG    0      0        0 eth0
	192.169.30.1    0.0.0.0         255.255.255.255 UH    0      0        0 eth0
	192.169.31.0    0.0.0.0         255.255.255.0   U     1      0        0 eth0
```

哟呵,网关地址 192.169.30.1 Metric 0

但是我实际的获取到的IP却是 192.169.31.23 Metric 1

这里说实话我是不太理解的,相当于我实际的地址到网关还还隔了一个路由器,咦?

那我就是ping一下:

```bash
$   ping 192.169.30.1
	PING 192.169.30.1 (192.169.30.1): 56 data bytes
	64 bytes from 192.169.30.1: seq=0 ttl=64 time=0.462 ms
	64 bytes from 192.169.30.1: seq=1 ttl=64 time=0.523 ms
	64 bytes from 192.169.30.1: seq=2 ttl=64 time=0.532 ms
	64 bytes from 192.169.30.1: seq=3 ttl=64 time=0.402 ms
```

额,ttl=64就是直连的嘛.

好吧,能ping通,那看看开了什么端口.

```bash
	Discovered open port 80/tcp on 192.169.30.1
	Discovered open port 22/tcp on 192.169.30.1
	Discovered open port 53/tcp on 192.169.30.1
```

80应该是什么后台管理

22这个可能和这个后台管理密码一样.

至于这个53端口DNS解析,大家都知道.

我试了下一些常用的初始化帐号密码和弱口令,没有成功.有些少许的失落.

最后还剩一个53端口,记得以前电信还是移动没有封DNS解析,可以通过这个端口免费上网呢,那都还是2.5G年代了.

好吧,那用网络有没有连接的域名来试试


```bash
$ nslookup www.baidu.com
```

哈哈哈,居然能通,能解析呢.
(udp 53)

```bash
$ telnet 4.4.2.2 53
```
yes,也能连接.
(tcp 53)

tcp/udp 53端口全开放,看来上网问题解决了.

大神们估计已经知道怎么办了.那么你如果不知道怎么办那么小弟现在就把这个方法告诉你(大神别见笑).

		  1                      2                3
	 你的房间(RJ-45)  ----->  房东路由器  ---->  DNS服务器
	      
房东的路由器会对所有接入的设备(电脑,路由器)分发一个IP,于是在内网其实你的电脑和房东的路由器其实已经可以通信了.

上不了网的原因是,房东在路由器上做一些规则(也可以叫防火墙),只针对拥有PPPOE帐号拨号的人,才允许他们上网.

但是我们传统上网都是需要做[DNS解析](https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F)的吧?

但是呢房东没有对这个进行白名单处理.任何人都可以做DNS解析.

好吧此处的重点就是,假设我在外网有一个服务器开放了53端口,然后我所有的数据包,都通过这53的端口发送,我是不是有可以上网了呢??

好吧,这些都成立了,那么下面就开始干活了:

 - 外网的一个服务器并可以使用的53端口(SS服务端使用).
 - 接入客户端可以连接53端口.


 我们姑且把这种连接叫做建立一种管道吧(叫虚拟专用网/隧道也可以).
 
 外网服务器我用的是xx云的服务器,上面搭建一个shadowsocks服务
 我的接入端我选择一个支持openwrt的路由器,上面搭一个shadowsocks透明代理.
 下面就是一些具体的配置惹:

 ### 服务端:
 
 ```bash
$ wget  https://raw.githubusercontent.com/wildcoder-me/stackscript/master/stackscript.sh
	 chmod a+x stackscript.sh
	 ./stackscript.sh
	 cat /etc/shadowsocks.json
 ```
 你没看错,服务器就这么容易搭建.
  
 ps:这个脚本是我fork [clowwindy](https://github.com/clowwindy)的一个仓库
 
 /etc/shadowsocks.json 这里就是你的ss服务器的相关信息啦.客户端配置需要这些的.
 好,下面给你们看看(当然这个不是我真实服务器信息).

 ```bash
	{
    "server":"0.0.0.0",
    "server_port":2462,
    "local_port":8108,
    "password":"11111111111111",
    "timeout":300,
    "method":"aes-256-cfb"
	}

 ```
 
 
 ### openwrt:
 
 需要安装这个包shadowsocks-libev,可以自己去下对应版本的包,也可以自己编译啦.
 然后就配置一个透明代理.
 
 ```bash
 $  vi /etc/init.d/shadowsocks
 ```
 
 主要是配置ss-redir这个透明代理,使用的
 
 ```bash 
	CONFIG_JP=/etc/shadowsocks-jp.json     
	start() {                           
        service_start /usr/bin/ss-redir -c $CONFIG -b 192.168.71.1 
	}                                                                                                                                                                               
	stop() {
        service_stop /usr/bin/ss-redir 
	} 
 
 ```
 
 ```
	iptables -t nat -N REDSOCKS2
	iptables -t nat -A PREROUTING -i br-lan -p tcp -j REDSOCKS2
	iptables -t nat -A PREROUTING -i br-lan -p udp -j REDSOCKS2

	#这个ip网段的地址不进行转发
	iptables -t nat -A REDSOCKS2 -d 127.0.0.0/8 -j RETURN
	iptables -t nat -A REDSOCKS2 -d 192.168.0.0/16 -j RETURN
	iptables -t nat -A REDSOCKS2 -d 10.8.0.0/16 -j RETURN
	iptables -t nat -A REDSOCKS2 -d 224.0.0.0/4 -j RETURN
	iptables -t nat -A REDSOCKS2 -d 240.0.0.0/4 -j RETURN
	
	#转发对应的透明代理的客户端的端口
	#--to-ports 这个地方是你上面shadowsocks.json的local_port
	iptables -t nat -A REDSOCKS2 -p tcp -j REDIRECT --to-ports 8107
	iptables -t nat -A REDSOCKS2 -p udp -j REDIRECT --to-ports 8107
 ``` 
 
好吧,这些基本上都搞定了.你可以愉快的上网了.

对于用了我上面说的方法,而引发的纠纷,小弟可是承担不起哦.

姿势补充: 其实还有以来ICMP包来穿透防火墙啦,其原理也差不多,只是用的协议不一样,当然你也需要有支持这些协议的服务,就像上面的shadowsocks

以上仅供学习,别拿去干坏事哦。
