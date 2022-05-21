---
title: openwrt简单的dns方案
date: 2016-10-03 14:05:00
tag: openwrt
---

### 准备

1. 环境 

```bash
    NETGEAR WNDR3700v4
    OpenWrt Chaos Calmer 15.05.1 / LuCI 15.05-149-g0d8bbd2 Release (git-15.363.78009-956be55)

```

2. 准备软件
   
```bash 
 dnscrypt-proxy 
```

### 安装 dnscrypt-proxy

```bash
$ opkg update
$ opkg install dnscrypt-proxy
```

简单不？
     
### 配置/etc/config/dnscrypt-proxy


```bash
config dnscrypt-proxy
	option address '127.0.0.1'
	option port '5553'
	# option resolver 'opendns'
	option resolvers_list '/usr/share/dnscrypt-proxy/dnscrypt-resolvers.csv'
```

开机运行&启动

```bash
/etc/init.d/dnscrypt-proxy enable
/etc/init.d/dnscrypt-proxy start
```

>一般openwrt 会使用dnsmasq作为dns和dhcp服务，所以这里也得该
编辑/etc/config/dhcp

```bash
config dnsmasq
	option domainneeded '1'
	option boguspriv '1'
	option filterwin2k '1'
	option localise_queries '1'
	option rebind_protection '1'
	option rebind_localhost '1'
	option local '/lan/'
	option domain 'lan'
	option expandhosts '1'
	option nonegcache '0'
	option authoritative '1'
	option readethers '1'
	option leasefile '/tmp/dhcp.leases'
	#option resolvfile '/tmp/resolv.conf.auto'  
	option noresolv  1
    	list server	'127.0.0.1#5553'
    	list server	'/pool.ntp.org/208.67.222.222'
```

- option resolvfile 这个是ISP的解析信息,污染源
- option noresolv 禁止使用这个 /etc/resolv.conf 文件的dns服务列表
- list server 使用上面我们定义的ndscrypt服务
- list server /pool.ntp.org/208.67.222.222 因为DNScrypt需要准确的时间，所以需要这个时间服务器

配置基本上完成了，重启下dnsmasq，客户端需要刷新下dhcp信息,如果不知道就重启机器是最简单的啦。


### 日志

```
$ logread
```


```bash
Mon Oct  3 13:40:16 2016 daemon.notice dnscrypt-proxy[1692]: Proxying from 127.0.0.1:5553 to 222.222.220.220:443
Mon Oct  3 13:46:40 2016 daemon.info hostapd: wlan0: STA 11:11:11:11:18:60 WPA: group key handshake completed (RSN)
Mon Oct  3 13:46:40 2016 daemon.info hostapd: wlan0: STA 11:11:11:11:18 WPA: group key handshake completed (RSN)
Mon Oct  3 13:46:40 2016 daemon.info hostapd: wlan0: STA 11:11:11:11:18 WPA: group key handshake completed (RSN)
Mon Oct  3 13:46:40 2016 daemon.info hostapd: wlan0: STA 11:11:11:11:18 WPA: group key handshake completed (RSN)
Mon Oct  3 13:56:40 2016 daemon.info hostapd: wlan0: STA 11:11:11:11:18 WPA: group key handshake completed (RSN)
Mon Oct  3 13:56:40 2016 daemon.info hostapd: wlan0: STA 11:11:11:11:18 WPA: group key handshake completed (RSN)
Mon Oct  3 13:56:40 2016 daemon.info hostapd: wlan0: STA 11:11:11:11:18 WPA: group key handshake completed (RSN)
Mon Oct  3 13:56:40 2016 daemon.info hostapd: wlan0: STA 11:11:11:11:18 WPA: group key handshake completed (RSN)

```

>有不足或者不正确的地方还希望指点。

设备支持列表[Openwrt](https://wiki.openwrt.org/toh/start)
参考[Openwrt](https://wiki.openwrt.org/inbox/dnscrypt)
参考[DNSCrypt](https://dnscrypt.org/#public-resolvers)