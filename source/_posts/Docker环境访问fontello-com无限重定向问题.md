title: Docker环境访问fontello.com无限重定向问题
author: 大丈夫没问题
tags:
  - linux
  - docker
  - fontello
categories:
  - linux
date: 2019-06-30 20:59:00
---
前阵子要把公司几个年代比较久远的前端项目迁移到新的jenkins环境，由于这些项目打包依赖库的版本比较老，要配置好环境会比较花时间，以后再迁移环境的话还要在搭建一遍，于是就想着用docker来打包项目。


本地搭建好docker环境后，测试打包的时候发现项目从fontello.com下载字体是老是会报一个超出最大重定向次数的异常，ssh到docker环境发现 `wget -vdS fontello.com` 返回的Response状态码一直是302，而本机环境和服务器都没这问题，docker环境`wget`其他网址也没有问题，一开始以为是fontello网站配置的问题，今天突然冒出一个想法会不会是我docker环境配置了代理的原因，于是将 `fontello.com` 域名加到了代理白名单里，发现就没有重定向异常了，字体文件能正常下载了。

这个问题的根本原因很有可能是docker代理功能和 `fontello.com` 两边的问题，在这里记一下，以防下次再遇到类似问题。