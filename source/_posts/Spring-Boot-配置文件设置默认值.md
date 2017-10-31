title: Spring Boot YAML配置文件设置字段默认值
author: Gang Chen
tags:
  - java
  - spring
  - yaml
categories:
  - Java
date: 2017-10-30 21:19:00
---
SpringBoot中application.yaml如何设置配置项默认值一直困扰了我很久,官方文档也没有写如何设置默认值,今天有时间研究了下,发现只要加个冒号就可以了,如`${配置名:默认值}`.

## application.yml
```
appconfig:
  fileServerAccessKey: ${FILE_SERVER_ACCESS_KEY:default_access_key}
  ...
```

## 配置文件类
```
@Component
@ConfigurationProperties(prefix = "appconfig")
class Config {
    String fileServerSecretKey
    ...
}
```
如果项目未设置`FILE_SERVER_ACCESS_KEY`环境变量,则fileServerSecretKey字段会以`default_access_key`为默认值.

研究了下源码发现 [PropertyPlaceholderHelper.java#L147](https://github.com/spring-projects/spring-framework/blob/49787493a6ca3726df62c4aff85023e8fa589ed3/spring-core/src/main/java/org/springframework/util/PropertyPlaceholderHelper.java#L147)这个类先会检查配置值中是否包含`valueSeparator`这个字符串,如果有,则会用`valueSeparator`来分割字符,从中取出默认值,而`valueSeparator`默认值就是`:`. 

