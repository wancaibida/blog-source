title: Openwrt下dnsmasq配置
author: 大丈夫没问题
tags:
  - dnsmasq
  - dns
  - openwrt
categories:
  - linux
date: 2020-02-17 21:42:00
---
## 配置文件

`dnsmasq.conf`

```
conf-dir=/root/dnsmasq.d/
//设置DNS缓存时间
min-cache-ttl=3600
//缓存数量
cache-size=1024
//重新加载后清空缓存
clear-on-reload
```

## 配置说明

### 转发域名解析

解析海外域名需要将dns解析请求转发到上游的无污染的dns服务器，配置文件 `/root/dnsmas.d/not_china.conf`
```
server=/google.com/127.0.0.1#5353
...
```

国内域名直接用国内的dns服务器，`china.conf`

```
server=/baidu.com/221.6.4.66
...
```

### 缓存域名解析

海外域名的解析时间一般会比较长，频繁的去请求上游DNS服务器既浪费时间又无意义，可以通过设置dnsmasq缓存来加快解析，减少请求上游DNS服务器的频率。

```
//设置DNS缓存时间
min-cache-ttl=3600
//缓存数量,最多10000
cache-size=9999
```

如果你不知道缓存数量应该设置为多少，可以通过下面命令查看dnsmasq的域名请求数作为参考：

`killall -s USR1 dnsmasq`

### 关于 `no-resolv` 配置

在不打开`no-resolv`的情况下，dnsmasq会使用ISP提供的dns服务器作为默认的服务器，比如 `xx.com`域名既不在 `not_china.conf`又不在`china.conf`中，dnsmasq就会用ISP的dns服务器来解析这个域名。

如果打开了`no-resolv`，同时又不设置`resolv-file`的话，dnsmasq就会找不到默认的dns服务器来解析`xx.com`域名，如果你的代理服务器正好属于这类域名，将导致你无法连接到你的服务器。

