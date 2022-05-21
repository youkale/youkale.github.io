---
title: Ngrok 编译的坑
date: 2016-06-01 17:57:00
tag: golang
---

### ngrok 是用go语言实现的，编译过程碰到不少坑

> 主要碰到的坑就是依赖错误，以及依赖的包被伟大的GFW给挡住了。
最好是用天朝外的VPS make release-server release-client 

修改日志src/ngrok/log/logger.go中的import部分改成

```bash
import (
  log "github.com/alecthomas/log4go"
  "fmt"
)

```

> 其他的跟网上的差不多了。
 下面就把依赖好的包打包发上来，以及编译的方法给大家说下：
 这个包是已经包含好了依赖，拿到包后，确认你的环境，如果你要编译客户端需要go 1.6 否则编译会通不过

- 安装依赖 

```bash
	sudo apt-get install build-essential golang mercurial git 
```

- 解压

```bash
 tar -zxv -f  ngrok.pkg.tar.gz 
```

- 生成证书

```bash
    openssl genrsa -out base.key 2048
    openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=ngrok.youkele.com" -out base.pem
    openssl genrsa -out server.key 2048
    openssl req -new -key server.key -subj "/CN=ngrok.youkele.com" -out server.csr
    openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt
    cp base.pem assets/client/tls/ngrokroot.crt

```

- 编译服务器

```bash
    make release-server
```

- 编译客户端 (go >= 1.6)

```bash
#linux
make release-client
#windows
GOOS=windows GOARCH=amd64 make release-client
#mac
GOOS=darwin GOARCH=amd64 make release-client
```

> 编译前如果make clean了，那记得把证书再copy一份cp base.pem assets/client/tls/ngrokroot.crt

### 运行

 - 服务器

```bash
 /usr/local/src/ngrok/bin/ngrokd -tlsKey=/usr/local/src/ngrok/server.key -tlsCrt=/usr/local/src/ngrok/server.crt -domain="ngrok.youkele.com" -httpAddr=":8090" -httpsAddr=":4433"
```

- 客户端

ngrok.cfg

```bash
    server_addr: "ngrok.youkele.com:4443"
    trust_host_root_certs: false
```

- 客户端 启动

```bash
	./ngrok -config=./ngrok.cfg -subdomain=test 8080
	./ngrok -h 可以看帮助
```

- 配置nginx 

```bash
server {
    listen 80;
    server_name *.ngrok.xxx.xxx;
    access_log /var/log/nginx/xxx.xxx.log;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host  $http_host:8090;
        proxy_set_header X-Nginx-Proxy true;
        proxy_set_header Connection "";
        proxy_pass      http://127.0.0.1:8090;

    }
}

```
