title: Safari can't establish a secure connection 解决办法
author: 大丈夫没问题
tags:
  - safari
categories:
  - linux
date: 2019-10-23 21:22:00
---
公司测试用的是6.x版本的Safari，在测试时发现无法打开项目网站，提示`Safari can't establish a secure connection`错误，测试了下其他网站也只有一部分能打开。

一开始以为是系统配置的问题，尝试了下网上的解决方案如：修改DNS服务器地址，信任证书等，发现还是无法打开项目网站。后来发现Safari 6.x是不支持TLS 1.2版本的，而我们项目用的正好是1.2版本的TLS。

解决办法：

1. 升级Safari版本
2. 让项目支持低版本的TLS


参考：
* https://www.projectdatasphere.org/projectdatasphere/html/tls/faq
* https://www.howsmyssl.com/