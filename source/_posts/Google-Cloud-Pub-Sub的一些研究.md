title: Google Cloud Pub Sub的一些研究
author: 大丈夫没问题
tags:
  - google
  - cloud
  - pub sub
categories:
  - linux
date: 2019-11-11 21:22:00
---
* 多个客户端可以使用一个`subscription`，每个客户端只会收到一部分的消息：

> Multiple subscribers can make pull calls to the same "shared" subscription. Each subscriber will receive a subset of the messages
出处：[https://cloud.google.com/pubsub/docs/subscriber#push-subscription](https://cloud.google.com/pubsub/docs/subscriber#push-subscription)


* 对于一些处理比较耗时的消息，客户端有后台任务会自动更新ack 
deadline，详情可见[extendDeadlines](https://github.com/googleapis/google-cloud-java/blob/v0.99.0/google-cloud-clients/google-cloud-pubsub/src/main/java/com/google/cloud/pubsub/v1/MessageDispatcher.java#L400)方法。


* 关于`Message retention duration`

> By default, a message that cannot be delivered within the maximum retention time of 7 days is deleted and is no longer accessible. This typically happens when subscribers do not keep up with the flow of messages. Note that you can configure message retention duration (the range is from 10 minutes to 7 days).

意思就是如果消息在`Message retention duration`时间内没有被处理完成的话，这条消息就会删除并且再也访问不到。

比如程序的BUG导致消息一直无法被处理，经过 `Message retention duration` 后这条消息就再也收不到了，会导致数据丢失。