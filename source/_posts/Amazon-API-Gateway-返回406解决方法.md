title: Amazon API Gateway 返回406解决方法
author: Gang Chen
tags:
  - aws
categories:
  - linux
date: 2018-03-28 21:40:00
---
在Integration Request里添加值为空字符串`''`的`Accept`header即可

参考：[aws-api-gateway-returns-http-406](https://stackoverflow.com/questions/42106605/aws-api-gateway-returns-http-406)
