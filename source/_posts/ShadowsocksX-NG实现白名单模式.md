title: ShadowsocksX-NG实现白名单模式
author: 大丈夫没问题
tags:
  - ss
categories: []
date: 2018-09-04 10:39:00
---
ShadowsocksX-NG `Proxy Auto Configure Mode`使用的是黑名单模式，默认直连，符合规则的走代理。要实现白名单模式，默认代理，符合规则的直连，有一种hack方法：

* 找到程序文件夹下的abp.js文件，路径一般为`/Applications/ShadowsocksX-NG.app/Contents/Resources/abp.js`

* 修改`FindProxyForURL`方法，将`direct,proxy`对调，改成下面代码中的样子

  ```
  function FindProxyForURL(url, host) {
    if (defaultMatcher.matchesAny(url, host) instanceof BlockingFilter) {
      return direct;
    }
    return proxy;
  }
  ```
* 重启程序
* 修改程序`preferences`里的`GFW List url`，改成白名单规则
* 然后点击`Update PAC from GFW List`更新pac文件即可