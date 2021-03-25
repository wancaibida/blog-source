title: MIUI系统下Tasker保持后台常驻的方法
author: 大丈夫没问题
tags:
  - miui
  - tasker
  - android
categories:
  - linux
date: 2021-03-25 22:48:00
---
之前一直是用苹果的短信同步功能来同步短信的，期间老是出现一些莫名其妙不能同步短信的问题，最近实在是受不了这问题了，打算换到用android备用机来转发短信。

一开始用的是ifttt来同步短信的，一切运行正常，唯一的缺陷是iftt同步短信会有大概30秒左右时间的延时，有点接受不了。
后来开始用tasker来转发短信，尝试了下效果比ifttt好，几乎没有延时，但遇到一个问题就是tasker不能常驻后台，参考了官网的方案也没有解决这问题，研究了两天发现还有两个地方忘记了设置，在这里记录一下。

测试环境 MIUI 12.0.9

## MIUI设置

### 将Tasker加到白名单
手机管家 -> 优化加速 -> 设置 -> 锁定任务 -> 将Tasker添加到已锁定任务

![1.png](/images/1.png)
![2.png](/images/2.png)
![3.png](/images/3.png)
![4.png](/images/4.png)

### 关闭智能场景省电


手机管家 -> 省电与电池 -> 场景配置 -> 睡眠模式 -> 关闭

![a.png](/images/a.png)
![b.png](/images/b.png)
![c.png](/images/c.png)
![d.png](/images/d.png)


## 其他设置

* [Why doesn't Tasker work in the background on my device?](https://tasker.joaoapps.com/userguide/en/faqs/faq-problem.html#00)


