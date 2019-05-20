title: Tomcat传递环境变量
categories:
  - java
  - ''
tags:
  - tomcat
date: 2017-06-13 08:21:00
---
1. 在Tomcat bin目录下新建setenv.sh, 编辑setenv.sh,添加如下语句

```
export JAVA_OPTS="-DKEY1=VALUE1 -DKEY2=VALUE2 -DKEY3=VALUE3"
```

2. Tomcat启动时会自动加载setenv.sh

3. 通过如下代码中获取环境变量

```
System.getProperty("KEY1")
```