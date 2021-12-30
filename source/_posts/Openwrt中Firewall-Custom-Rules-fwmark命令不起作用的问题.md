title: Openwrt中Firewall - Custom Rules fwmark命令不起作用的问题
author: 大丈夫没问题
tags:
  - openwrt
  - linux
categories:
  - linux
date: 2021-12-30 20:58:00
---
路由器设置了透明代理, 但是每次重启路由器后代理就是失效了, 发现Custom Rules中`ip rule add fwmark 1 table 100` 这个命令不知道啥原因在重启后会被清除, 导致代理失败.

解决办法就是编辑 `/etc/config/network`

添加以下内容
```
config rule
	option mark   '0x1'
	option lookup '100'

config route
	option interface 'loopback'
	option target '0.0.0.0'
	option netmask '0.0.0.0'
	option table '100'
	option type 'local'
```

代替Custom Rules的这两条命令:

```
ip rule add fwmark 1 table 100
ip route add local 0.0.0.0/0 dev lo table 100
```

参考:
- https://forum.openwrt.org/t/equivalent-of-ip-route-rule-to-be-saved-in-etc-config-network/76593
- https://openwrt.org/docs/guide-user/network/ucicheatsheet#ipv4_routes