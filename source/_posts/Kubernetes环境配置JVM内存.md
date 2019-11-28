title: Kubernetes环境配置JVM内存
author: 大丈夫没问题
tags:
  - java
  - kubernetes
  - k8s
  - docker
  - jvm
categories:
  - linux
date: 2019-11-28 22:00:00
---
我们知道JVM在docker容器环境中是无法正确检测到可用内存的，最近正好遇到了一个与之相关的问题，在此记录一下。

遇到问题的项目技术栈为JDK 8 + Spring Boot + Tomcat，部署在docker环境。项目运行过程中出现了`java.lang.OutOfMemoryError: Java heap space`异常，当时项目的部署文件如下：

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: api-deployment
  labels:
    app: api
spec:
  serviceName: api-app
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      terminationGracePeriodSeconds: 30
      containers:
        - image: ...
          imagePullPolicy: "Always"
          name: api
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 300
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 5
          securityContext:
            capabilities:
              add:
                - SYS_PTRACE 
          envFrom:
            - secretRef:
                name: secret

```
问题应该出在k8s内存设置与JVM的配置这边，网上查询资料后发现tomcat可以通过`CATALINA_OPTS`环境变量来设置JVM参数，`UseCGroupMemoryLimitForHeap` 可以让JVM自动检测容器的可用内存，`MaxRAMFraction` 为容器内存和堆内存的比例，比如容器内存为2G，MaxRAMFraction为2，则最大堆内存为2G/2=1G，这里将`MaxRAMFraction`设置为2比较安全，设置了这两个参数后，JVM就能通过检测容器的内存来自动调整堆内存大小，不用再显示设置堆内存了。


更新后的配置文件里加了如下代码：

```
...
          env:
              - name: CATALINA_OPTS
                value: "-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=2"
          resources:
            requests:
              memory: "512Mi"
```

项目运行一段时间后发现问题依旧，研究了下`UseCGroupMemoryLimitForHeap`参数，发现它是通过读取系统`/sys/fs/cgroup/memory/memory.limit_in_bytes`文件来检测内存的，登录到容器里查看了下该文件，发现里面是一个很大的值：`9223372036854771712`，等于没有内存限制，查了下资料发现这个字段是通过k8s文件中的`resources->limits`的这个属性来配置的，更新文件，加了如下代码：

```
            limits:
              memory: "2048Mi"
```

观察一段时间后内存就没再溢出，最终完整配置文件如下：

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: api-deployment
  labels:
    app: api
spec:
  serviceName: api-app
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      terminationGracePeriodSeconds: 30
      containers:
        - image: ...
          imagePullPolicy: "Always"
          name: api
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 300
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 5
          securityContext:
            capabilities:
              add:
                - SYS_PTRACE 
          env:
              - name: CATALINA_OPTS
                value: "-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=2"
          envFrom:
            - secretRef:
                name: secret
          resources:
            requests:
              memory: "512Mi"
            limits:
              memory: "2048Mi"
```

## 参考

* https://github.com/docker-library/tomcat/issues/157
* https://blog.csanchez.org/2017/05/31/running-a-jvm-in-a-container-without-getting-killed/
* https://www.logicbig.com/how-to/java-command/jvm-option-list.html
* https://medium.com/adorsys/jvm-memory-settings-in-a-container-environment-64b0840e1d9e
* https://stackoverflow.com/a/53826135/1776024