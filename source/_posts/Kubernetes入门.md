title: Kubernetes入门
author: Gang Chen
tags:
  - docker
  - kubernetes
categories:
  - docker
  - ''
date: 2017-11-20 22:09:00
---
# Kubernetes入门

## Kubernetes是什么
Kubernetes是用来自动化部署,扩展,管理容器应用的工具.
## Kubernetes架构
Kubernetes主要由Master和Node两部分组成.
![Architecture](http://static.w2x.me/0.png)
### Master
Kubernetes集群包含至少一个Master节点和多个Node节点.Master节点主要用来暴露API,调度部署和管理整个集群.
### Node
Node是工作节点,Node可能是物理机器也可能是虚拟机,每个节点上都提供了容器的运行环境,比如说Docker,rkt.
## Kubernetes对象
Kubernetes对象持久化在kubernetes系统中,Kubernetes用这些对象来描述集群的状态,比如:

* 部署的应用
* 应用的行为,比如重启,升级策略
* 应用程序可用的资源

### 描述Kubernetes对象
我们通常会通过.yaml来描述Kuberntes对象,Kubernetes客户端会将yaml文件转成JSON来调用API.
例:

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
我们可以通过下面的命令来创建Kubernetes对象:

```
kubectl create -f docs/user-guide/nginx-deployment.yaml --record
```

#### 字段说明
在yaml文件中,你需要声明以下字段:

* apiVersion - 调用的API版本
* kind - 对象类型
* metadata - 用于查找对象的字段,比如名称,UID,命名空间

一些常见的kubernetes对象有:

* Pod
* Deployments
* Service
* Ingress

### Pod对象
Pod对象是最小的布署单元,包含了一个或多个容器.通常我们不会单独使用Pod.
Controller对象可创建多个Pod,常风的Controller对象有:

* Deployment
* StatefulSet
* DaemonSet

### Deployments对象
Deployments对象可以创建Pod,在Deployments对象中你只需要描述你需要的状态,Deployments对象会将当前状态改变成你所需要的状态.

 例:

```
 apiVersion: apps/v1beta1
 kind: Deployment
 metadata:
   name: nginx-deployment
 spec:
   replicas: 3
   template:
     metadata:
       labels:
         app: nginx
     spec:
       containers:
       - name: nginx
         image: nginx:1.7.9
         ports:
         - containerPort: 80
```
上面的描述文件创建了3个运行Nginx的Pod对象.

通过下面命令来部署和查看Deployments对象:

```
kubectl create -f docs/user-guide/nginx-deployment.yaml --record
kubectl get deployment
```

### Service对象
Service对象将Pod对象抽像成一种服务,并定义了访问这些Pod对象的策略,类似于负载均衡(Loadbalancer).Service对象会通过选择器来选择Pod对象.

例:

```
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
### Ingress对象
Ingress对象定义了外部请求如何转发到Service,类似于反向代理.
例:

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bar
        backend:
          serviceName: s2
          servicePort: 80
```
上面的描述文件定义了Service的访问规则,foo上下文路径会被转发到名为s1的Service,bar上下文路径转发到名为s2的Service.

简而言之:

* Deployments定义了怎样部署应用(应用image,部署多少实例,启动策略等);
* Service定义了如何抽像Deployments为服务,类似于负载均衡;
* Ingress定义了请求如何转发到Service,类似反向代理,如nginx.






