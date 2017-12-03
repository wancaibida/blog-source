title: 记一个Grails JSON Views的Bug
author: Gang Chen
tags:
  - Grails
  - JSON
categories:
  - Java
date: 2017-12-03 15:25:00
---
最近项目用到了3.2版本的Grails，这个版本中引入了一个新特性JSON views，主要作用是将JSON返回内容视为一种视图,类似于GSP。其好处就是可以在视图层定义返回json的格式，而且可以定义bean的JSON模版，比较灵活。

项目中有一个名为`QueryResult`的类，并设置了`QueryResult`类的模版，名为`_queryresult.gson`。

文件目录如下：

```
bean
    package...
        QueryResult.groovy
views
    queryResult
        _queryresult.gson
```
Controller代码：

```
def list(xxx){
    new QueryResult(xxx)
}
```

项目在本地开发时没有什么问题，但部署到生产环境时，这个方法返回内空始终为空。

项目在本地开发时是在内置的tomcat运行的，而在生产环境中项目是打包成war文件部署在Linux下的Tomcat。猜测问题可能是项目运行方式不同导致。

调试后发现了如下代码：

```
WritableScriptTemplate template
if (Environment.isDevelopmentEnvironmentAvailable()) {
    template = attemptResolvePath(path)
    if (template == null) {
        template = attemptResolveClass(path)
    }
} else {
    template = attemptResolveClass(path)
    if (template == null) {
        template = attemptResolvePath(path)
    }
}
if (template == null) {
    template = NULL_ENTRY
}
```
代码的基本意思是，如果是在开发环境下，模版内容优先通过文件路径来查找，如果没有找到则通过类名来查找，而在生产环境正好反过来。

模版文件`_queryresult.gson`编译后会生成名为`xxx_queryResult__queryresult_gson.class`的java类。

JSON Views插件会通过一系列约定的命名方式来找所需要的视图，最终会查找名为`xxx_queryResult__queryResult_gson.class`的java类（注意：第二个queryResult中的R是大写的），前面说到开发环境下会先通过文件名称来查找，即

```
//实际文件名为：xxx_queryResult__queryresult_gson.class
def templateFile = new File('xxx_queryResult__queryResult_gson.class')
if(templateFile.exist()){
    return templateFile;
}
```
，如果文件存在，则返回。可实际的class文件名为`xxx_queryResult__queryresult_gson.class`，但我开发环境的文件系统是**大小写不敏感**的！！，所以对于系统来说`xxx_queryResult__queryresult_gson.class`和`xxx_queryResult__queryResult_gson.class`是同样的文件名，`exist`方法最终是通过系统的API来寻找文件的，所以`templateFile.exist()`这句返回结果为真。

在生产环境是下是优先通过类名来查找的，要查找的类名为`xxx_queryResult__queryResult_gson.class`，而实际的类名为`xxx_queryResult__queryresult_gson.class`，而java类名是区分大小写的，所以会导致类找不到。

一旦通过类名查找不成功，则会通过文件路径来查，而Linux的文件系统是**大小写敏感**的，所以会无法找到名为`xxx_queryResult__queryResult_gson.class`的文件。

最终把模版文件名重命名为`_queryResult.gson`，接口就运行正常了。

总结：
严格来说这个BUG是文件系统对大小写处理的差异导制的，所以在开发时要考虑到文件系统对大小写处理不一致的问题。