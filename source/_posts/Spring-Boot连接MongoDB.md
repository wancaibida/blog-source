title: Spring Boot连接MongoDB
author: 大丈夫没问题
tags:
  - mongodb
  - database
  - java
  - spring
categories:
  - java
date: 2019-04-18 22:36:00
---
最近在做的项目用到了Mongo DB，发现在开启用户认证的情况下，用URI的方式连接总是提示认证失败，只有在分别设置了`username ,password , database`等字段才能连接成功。

```
 spring:
     data:
         mongodb:
             authentication-database: ${MONGODB_AUTHENTICATION_DATABASE:admin}
             username: ${MONGODB_USERNAME:username}
             password: ${MONGODB_PASSWORD:password}
             database: ${MONGODB_DATABASE:database}
```

之前一直认为Spring Boot连接Mongo DB的方式和连JDBC差不多，今天抽空研究了下，发现URI中的数据库名其实是存储用户认证信息的数据库，实际数据库要通过`database`设定：


```
spring:
    data:
        mongodb:
            database: ${MONGODB_DATABASE:database}
            uri: ${MONGODB_URI:mongodb://username:password@localhost:27017/authentication_database}
```

[官方文档](<https://docs.mongodb.com/manual/reference/connection-string/>)：

> /database

> Optional. The name of the database to authenticate if the connection string includes authentication credentials in the form of `username:password@`. If `/database` is not specified and the connection string includes credentials, the driver will authenticate to the `admin` database. See also [`authSource`](https://docs.mongodb.com/manual/reference/connection-string/#urioption.authSource).

