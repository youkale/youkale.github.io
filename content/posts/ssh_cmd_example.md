+++
draft = false
date = 2022-06-11T12:32:29+08:00
title = "SSH常见用法"
description = "用SSH配置与映射"
slug = ""
authors = ["Youkale"]
tags = ["linux"]
categories = []
externalLink = ""
series = []
+++

分享一些常见用法，希望能在工作中能提高效率。

### 涵盖以下内容:

- .ssh/config 配置
- 常用映射
  - 远程->本地 `-L`
  - 本地->远程 `-R`
  - 动态代理 `-D`

- 其他ssh命令集
  - scp
  - ssh-copy-id
  - ssh-keygen

### .ssh/config配置简要说明

```shell
### ----------- ~/.ssh/config -----------
## 你配置的服务器名称，可根据自己的喜好配置, 我这里配置的是 server_1
Host server_1  																		
    ## ip 或者 domain, 可以访问的地址
    HostName 223.5.5.5 														
    ## 默认使用的用户名
    User ubuntu  																	
    ## 服务器端口
    Port 22  																			
		## rsa私钥地址
		IdentityFile ~/.ssh/server_rsa 
    ## 代理配置, 表示通过本地 socksv5 服务器进行代理
		ProxyCommand nc -X 5 -x 127.0.0.1:8106 %h %p  


### 接下来可以直接在终端下面这样使用: 
### ----------- 使用方法 -----------
### 使用上面配置的 server_1
~$ ssh server_1

```

- 跳板机例子:

```shell
### ----------- ~/.ssh/config -----------
Host jump
    HostName 223.5.5.5
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/jump_rsa
    
Host server
    HostName 1.1.1.1
    User root
    Port 22
    IdentityFile ~/.ssh/server_rsa
    ProxyCommand ssh -W %h:%p jump 
    ### 为什么用 `ssh -W` ？在没有安装netcat的情况可以尝试使用 -W 
    
### ----------- 使用方法 -----------
### 使用方法不变
~$ ssh server
```

 - Github例子

```shell
### ----------- ~/.ssh/config -----------
Host github.com
    HostName ssh.github.com
    Port 443
    IdentityFile ~/.ssh/github_rsa
    #ProxyCommand nc -X 5 -x 127.0.0.1:8108 %h %p
```



### 常用映射

- `-L  [bind_address:]port:host:hostport `  将远程端口映射到本地
  - 详细说明：

  
  > 通常用于将远程服务器上只对本地或内网开辟的服务映射到本地, 
  > 将远程服务器上mysql数据库映射到本地，其中 3307表示本地绑定的端口，192.168.0.11表示是 @server能访问到的一个内网地址
  > `ssh -N -L 3307:192.168.0.11:3306 root@sever`
  > 接下来就可以通过本地 `3307`端口访问到远程服务器上的mysql服务器了。
  
- `-R [bind_address:]port:host:hostport` 将本地服务暴露到远程服务器。
  
  - 详细说明：
  
  
  > 与`-L`相反，通常用于将本地或内网开辟的服务映射到远程。
  > `ssh -N -R 192.168.1.12:3307:192.168.2.11:3306 root@sever`
  > 将本地服务器`192.168.1.12:3307`上mysql数据库映射到远程，`127.0.0.1:3306`表示服务器对应的端口。 
  
- `-D [bind_address:]port` 在本地启用一个socksv5代理

  - 详细说明:

  
  >在某些特殊情况下，需要通过服务器的IP来访问某些内容，可以通过该参数启用一个本地代理
  >
  >`ssh -D 1080 -N server`
  >
  >接下来结合浏览器或者其他工具就能实现你的目的了。



更多内容可以通过`man ssh`进行查看， 感谢看到最后。

相关工具:

[explain-shell](https://explainshell.com) shell 执行计划，类似SQL的explain

[tldr](https://github.com/isacikgoz/tldr) 一个命令行的好帮手
