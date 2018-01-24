title: Intellij远程调试Tomcat
author: Gang Chen
tags:
  - intellij
  - tomcat
categories:
  - ide
  - ''
date: 2017-12-02 14:02:00
---
1. 在Tomcat下新建`setenv.sh`文件
2. 运行`chmod 755 setenv.sh`
3. 将下面的代码添加到`setenv.sh`文件中

    ```
    export CATALINA_OPTS="-agentlib:jdwp=transport=dt_socket,address=1043,server=y,suspend=n"

    ```
4. Intellij Run/Debug Configurations下新建Tomcat Server Remote
![Screen Shot 2017-12-02 at 13.53.42](http://static.w2x.me/2017-12-02-Screen Shot 2017-12-02 at 13.53.42.png)

5. 在新建的页面先择`Starup/Connection`标签,然后选择Debug选项
![Screen Shot 2017-12-02 at 13.56.20](http://static.w2x.me/2017-12-02-Screen Shot 2017-12-02 at 13.56.20.png)

6. 将`Port`的值修改为`1043`
![Screen Shot 2017-12-02 at 13.58.42](http://static.w2x.me/2017-12-02-Screen Shot 2017-12-02 at 13.58.42.png)

7. Tomcat启动后,Intellij即可远程Debug





