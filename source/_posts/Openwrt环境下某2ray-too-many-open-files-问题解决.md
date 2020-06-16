title: Openwrt环境下某2ray `too many open files`问题解决
author: 大丈夫没问题
tags:
  - openwrt
categories:
  - linux
date: 2020-06-01 21:07:00
---
`too many open files`大部分情况下是由于配置错误引起的。如果配置没有问题,可以在某2ray的启动脚本里加上这一句：


```
ulimit -SHn 65535
/usr/bin/2ray -config config.json
```

启动后可以用这个命令来看是否生效：

```
cat /proc/{2AY进程id}/limits
```


参考：
* https://blog.ihipop.info/2011/01/2053.html
* https://routeragency.com/2019/06/09/v2raye59ca8openwrte4b88be79a84e5ae89e8a385e983a8e7bdb2.html