title: Openwrt环境下某2ray `too many open files`问题解决
author: 大丈夫没问题
tags:
  - openwrt
categories:
  - linux
date: 2020-06-01 21:07:00
---
修改`/etc/sysctl.conf`文件，增加下面这一行:

```
fs.file-max = 65535
```

参考：
* https://routeragency.com/2019/06/09/v2raye59ca8openwrte4b88be79a84e5ae89e8a385e983a8e7bdb2.html