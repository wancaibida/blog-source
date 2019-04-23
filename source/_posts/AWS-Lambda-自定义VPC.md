title: AWS Lambda 自定义VPC
author: 大丈夫没问题
tags:
  - aws
  - lambda
  - linux
categories:
  - linux
date: 2019-04-23 22:11:00
---
AWS lamba 默认配置的VPC是能够访问外网的，如果要自定义VPC，同时能访问外网的话需要将lambda所在的子网挂在NAT网关下，同时这个NAT网关需要关联到一个有外网访问权限的子网下。

如图：

![AWS Lambda VPC](http://static.w2x.me/document_db_subnet.png)



参考：

[AWS Lambda: Enable Outgoing Internet Access within VPC](<https://medium.com/@philippholly/aws-lambda-enable-outgoing-internet-access-within-vpc-8dd250e11e12>)

