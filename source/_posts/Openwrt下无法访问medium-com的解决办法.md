title: Openwrt下无法访问medium.com的解决办法
author: 大丈夫没问题
tags:
  - openwrt
  - ipv6
  - ipv4
categories: []
date: 2020-02-09 21:57:00
---
家里路由器刷了Openwrt系统，在访问`medium.com` ,`nytimes.com`等网站时，会时不时出现连接重置错误，今天花时间研究了下，终于解决了这个困扰我很久的问题，在此记录下。

## 问题描述
* Chrome访问这些网站是会提示`ERR_CONNECTION_RESET`错误，反复刷新多次后又能打开网站。
* 用`wget -v`命令显示返回了ipv6地址并且会提示无法创建SSL连接错误。

## 问题原因

问题就出在ipv6上，openwrt默认会打开ipv6地址分配，这会导致电脑被分配到了一个ipv6地址，而chrome，safari在本机有ipv6的情况下，会优先访问这些网站的ipv6的地址，而路由器上的iptables只会对ipv4包进行转发，故访问这些网站的ipv6地址是无法被代理的。

## 解决办法

### 方法1

关闭路由器ipv6地址分配，这样在没有ipv6地址的情况下会访问这些网站的ipv4地址，这样就可以被代理到了。

### 方法2:
添加ip6tables规则，对ipv6也进行转发（没有尝试过）

## 参考
* https://superuser.com/questions/1131112/ipv4-only-wifi-in-an-ipv6-enabled-openwrt-router