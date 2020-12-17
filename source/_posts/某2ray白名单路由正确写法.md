title: 某2ray白名单路由正确写法
author: 大丈夫没问题
tags:
  - linux
categories:
  - linux
date: 2020-12-17 21:31:00
---
之前一直用的[Loyalsoldier](https://github.com/Loyalsoldier/v2ray-rules-dat)的路由规则文件，路由是这么写的：

```
"routing": {
    "rules": [
        {
            "type": "field",
            "outboundTag": "direct",
            "domain": [
                "geosite:private",
                "geosite:apple-cn",
                "geosite:cn"
            ]
        },
        {
            "type": "field",
            "outboundTag": "direct",
            "ip": [
                "geoip:private",
                "geoip:cn"
            ]
        }
    ]
}
```

最近发现`gstatic.com`域名无法访问了，研究发现`gstatic.com`域名是包含在`getsite:cn`里面的，正确写法如下：
```
"routing": {
  "rules": [
    {
      "type": "field",
      "outboundTag": "Direct",
      "domain": [
        "geosite:private",
        "geosite:tld-cn"
      ]
    },
    {
      "type": "field",
      "outboundTag": "Proxy",
      "domain": [
        "geosite:geolocation-!cn"
      ]
    },
    {
      "type": "field",
      "outboundTag": "Direct",
      "domain": [
        "geosite:cn"
      ]
    }
  ]
}

```