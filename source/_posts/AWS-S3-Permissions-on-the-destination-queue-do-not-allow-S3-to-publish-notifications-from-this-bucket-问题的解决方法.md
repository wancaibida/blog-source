title: >-
  AWS S3 'Permissions on the destination queue do not allow S3 to publish
  notifications from this bucket' 问题的解决方法
author: 大丈夫没问题
tags:
  - aws
  - linux
categories:
  - linux
date: 2019-03-20 00:55:00
---
在AWS平台给S3配置发送存储事件到SQS时遇到了如下错误提示：

>  Unable to validate the following destination configurations. Permissions on the destination queue do not allow S3 to publish notifications from this bucket.

在IAM那里捣鼓了半天，给S3加上了SQS相关的权限后依然提示错误信息。 正当毫无头绪的时发现SQS有个`Permission`标签，在这里配置相关权限后就可以解决错误提示了。

![Image](http://static.w2x.me/63817F26122AE59511F30628B2EB41A2.jpg)

原来SQS默认是不接受发送过来的事件的，印象当中之前是不需要配置的，也不知道这功能是啥时候加上去的，感觉AWS的权限系统有时太复杂了。。。

