title: ASUS 66u 搭建透明代理
author: 大丈夫没问题
tags: []
categories:
  - ss
date: 2018-09-20 16:57:00
---
## 安装软件

```shell
/usr/sbin/entware-setup.sh

opkg install bind-dig
opkg install shadowsocks-libev-ss-redir
opkg install shadowsocks-libev-ss-tunnel
opkg install libc libssp libev libmbedtls libpcre libpthread libsodium haveged zlib libopenssl
opkg install ipset4

//ac66u only
opkg install iptables
```

## 配置ss-redir服务

### 修改配置文件

1. `vim /opt/etc/init.d/S22shadowsocks`
2. 将`ss-local`改为`ss-redir`
3. 添加启动参数`-b 0.0.0.0`
4. 将`ENABLED`值改为 `yes`,变成开机启动

### 修改`shadowsocks.json`

修改服务器IP，端口，加密码方式。

## 配置dnsmasq

### 创建配置文件

* `vim /jffs/configs/dnsmasq.conf.add`
* 添加`conf-dir=/opt/etc/dnsmasq.d`

### 创建目录

* `mkdir /opt/etc/dnsmasq.d/`

* 下载规则文件

```shell
cd /opt/etc/dnsmasq.d/
curl -O https://cokebar.github.io/gfwlist2dnsmasq/dnsmasq_gfwlist.conf
```

## 配置ss-tunnel服务

ss-tunnel主要用于解决dns污染的问题，dnsmasq会将解析请求转发到ss-tunnel


### 创建开机启动脚本

* 转到目录`/jffs/scripts/`
* 创建名为比如`start-ss-tunnel.sh`文件，内容如下:

```shell
#!/bin/sh

/opt/bin/ss-tunnel -c /tmp/mnt/onmp/entware/etc/shadowsocks.json -l 5353 -L 8.8.8.8:53 -u > /dev/null 2>&1 &
```
创建`service-start`文件
```
chmod +x service-start
```
将`/jffs/scripts/start-ss-tunnel.sh` 加到`service-start`中

至此`ss-tunnel`就能开机启动了

## 配置iptables

### 下载chnroute.txt
```
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /jffs/configs/chnroute.txt

```

### 生成ipset集合
```
#!/bin/sh

wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /jffs/configs/chnroute.txt

ipset -N chnroute nethash

for ip in $(cat '/jffs/configs/chnroute.txt'); do
  ipset -A chnroute $ip
done


ipset --save > /jffs/configs/ipset.conf
```
因为`chnroute.txt`文件比较大,生成会比较慢,故将生成的ipset保存到`ipset.conf`中,每次启动时从`ipset.conf`中导入IP.

### 启动脚本

SS启动脚本`ss-up.sh`
```
#!/bin/sh

alias iptables='/opt/sbin/iptables'

ipset -R < /jffs/configs/ipset.conf

# ipset -N chnroute iphash

# for ip in $(cat '/jffs/scripts/chnroute.txt'); do
#  ipset -A chnroute $ip
# done

iptables -t nat -N SHADOWSOCKS
iptables -t mangle -N SHADOWSOCKS

# 直连服务器 IP
iptables -t nat -A SHADOWSOCKS -d [SS-SERVER-IP]/24 -j RETURN

# 允许连接保留地址
iptables -t nat -A SHADOWSOCKS -d 0.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 127.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 169.254.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 172.16.0.0/12 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 224.0.0.0/4 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 240.0.0.0/4 -j RETURN

# 直连中国 IP
iptables -t nat -A SHADOWSOCKS -p tcp -m set --match-set chnroute dst -j RETURN
iptables -t nat -A SHADOWSOCKS -p icmp -m set --match-set chnroute dst -j RETURN

# 重定向到 ss-redir 端口
iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports 1080
iptables -t nat -A SHADOWSOCKS -p udp -j REDIRECT --to-ports 1080
iptables -t nat -A OUTPUT -p tcp -j SHADOWSOCKS

# Apply the rules
iptables -t nat -A PREROUTING -p tcp -j SHADOWSOCKS
iptables -t mangle -A PREROUTING -j SHADOWSOCKS
```

将其中的`SS-SERVER-IP`修改成你的服务器地址

创建`post-mount`文件

```
chmod +x post-mount
```

将`ss-up.sh`加入

```shell
#!/bin/sh
/jffs/scripts/ss-up.sh
```

经测试只有将启动脚本加入到`post-mount`中对能开机启动，可能的原因是`ss-up.sh`中设及到挂载分区的访问。

### 停止脚本

```
#!/bin/sh
iptables -t nat -D OUTPUT -p icmp -j SHADOWSOCKS
iptables -t nat -D OUTPUT -p tcp -j SHADOWSOCKS
iptables -t nat -F SHADOWSOCKS
iptables -t nat -X SHADOWSOCKS
ipset -X chnroute
```

## 参考

* [shadowsocks-libev ss-nat](<https://github.com/shadowsocks/shadowsocks-libev/blob/master/doc/ss-nat.asciidoc>)
* [另一个ss-nat](<https://github.com/zfl9/ss-tproxy>)
* [U盘挂载](<https://zhih.me/format-Upan-partition/>)
* [gfwlist2dnsmasq](<https://github.com/cokebar/gfwlist2dnsmasq>)
* [Iptables 指南 1.1.19](http://www.path8.net/docs/iptables-tutorial_cn/iptables-tutorial-cn-1.1.19.html)
* [ss/ssr/v2ray/socks5 透明代理](https://www.zfl9.com/ss-redir.html)
* [https://www.shintaku.cc/posts/gfwlist/](https://www.shintaku.cc/posts/gfwlist/)
* [https://zzz.buzz/zh/gfw/2016/02/16/deploy-shadowsocks-on-routers](https://zzz.buzz/zh/gfw/2016/02/16/deploy-shadowsocks-on-routers/)
* [luci-app-shadowsocks](https://raw.githubusercontent.com/shadowsocks/luci-app-shadowsocks/master/files/root/usr/bin/ss-rules)
* [https://blog.bluerain.io/p/SS-Redir-For-Router.html](https://blog.bluerain.io/p/SS-Redir-For-Router.html)
* [transparent proxy base on ss, ipset, iptables, chinadns on asuswrt merlin](https://github.com/zw963/asuswrt-merlin-transparent-proxy)
* [利用shadowsocks打造局域网翻墙透明网关](https://medium.com/@oliviaqrs/%E5%88%A9%E7%94%A8shadowsocks%E6%89%93%E9%80%A0%E5%B1%80%E5%9F%9F%E7%BD%91%E7%BF%BB%E5%A2%99%E9%80%8F%E6%98%8E%E7%BD%91%E5%85%B3-fb82ccb2f729)
* [在 ArchLinux 上配置 shadowsocks + iptables + ipset 实现自动分流](https://typeblog.net/set-up-shadowsocks-with-iptables-and-ipset-on-archlinux/)
* [asuswrt-merlin/wiki/User-scripts](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts)