title: Mac OS配置Dnsmasq+Dnscrypt来解决DNS污染问题
author: 大丈夫没问题
tags:
  - mac
  - linux
  - dnsmasq
  - dns
categories:
  - linux
date: 2019-06-12 21:18:00
---
天朝的网络是越来越糟糕了，前阵子访问amazon s3域名居然返回了`127.0.0.1`。与其天天更新DNS列表，不如直接上DNS白名单模式了。

解决方案就是用`Dnsmasq+Dnscrypt`，`Dnsmasq`负责监听本地53端口，解析国内域名，将除了国内域名以外的解析请求转发到`Dnscrypt`监听的端口，`Dnscrypt`就收到的DNS解析请求转发到国外的DNS服务器。

## 安装
```
brew install dnsmasq
brew install dnscrypt-proxy
```

## 配置
### Dnsmasq

#### 编辑配置文件

```
vim /usr/local/etc/dnsmasq.conf

no-resolv
conf-dir=/usr/local/etc/dnsmasq.d
server=127.0.0.1#5300
```
dnsmasq会将DNS解析请求转发到dnscrypt监听的5300端口

#### 国内域名文件

执行下面脚本将中国域名DNS信息保存到dnsmasq文件夹
```
#!/bin/sh

/usr/bin/curl -o /usr/local/etc/dnsmasq.d/accelerated-domains.china.conf https://raw.githubusercontent.com/felixonmars/dnsmasq-china-list/master/accelerated-domains.china.conf
```
这里用的是电信的DNS服务器，如果你是联通的网络，你可以执行下面的脚本将配置文件中的DNS服务器换成联通的
```
sed -i -e 's/114.114.114.114/221.6.4.66/g' /usr/local/etc/dnsmasq.d/accelerated-domains.china.conf 
```

### Dnscrypt
Dnscryp配置 `vim /usr/local/etc/dnscrypt-proxy.toml`
```
listen_addresses = ['127.0.0.1:5300', '[::1]:5300']
fallback_resolver = '221.6.4.66:53'
server_names = ['google', 'cloudflare', 'cloudflare-ipv6']
```
这里本来想配5353端口的，但发现这个端口被chrome占用了。


## 启动服务
```
sudo brew services restart dnsmasq
sudo brew services restart dnscrypt-proxy
```
重启后将本机DNS指向127.0.0.1即可。


