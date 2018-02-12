title: Sl4j+Logback依赖配置
author: Gang Chen
tags:
  - java
  - logback
  - sl4j
  - gradle
categories:
  - java
date: 2018-02-12 21:29:00
---
```
    def sl4jVersion = '1.7.25'
    def logbackVersion = '1.2.3'

    compile group: 'ch.qos.logback', name: 'logback-core', version: logbackVersion
    compile group: 'ch.qos.logback', name: 'logback-classic', version: logbackVersion
    compile group: 'ch.qos.logback', name: 'logback-access', version: logbackVersion
    compile group: 'org.slf4j', name: 'slf4j-api', version: sl4jVersion
    compile group: 'org.slf4j', name: 'jcl-over-slf4j', version: sl4jVersion
    compile group: 'org.slf4j', name: 'log4j-over-slf4j', version: sl4jVersion

```