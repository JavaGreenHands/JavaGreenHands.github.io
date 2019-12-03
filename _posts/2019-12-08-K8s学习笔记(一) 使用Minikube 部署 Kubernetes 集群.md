---
date: 2019-12-08 18:00:00
layout: post
title: K8s学习笔记(一) 使用Minikube 部署 Kubernetes 集群
description: >-
    初始k8s
image: >-
  ../blogimg/k8s/k8s.jpeg
optimized_image: >-
  ../blogimg/k8s/k8s.jpeg
category: docker
tags:
  - docker
  - k8s
  - 运维
author: mrwhite
paginate: true
---

### K8s学习笔记(一) 使用Minikube 部署 Kubernetes 集群

#### K8s集群

Kubernetes将底层的计算资源连接在一起对外体现为一个高可用的计算机集群。Kubernetes将资源高度抽象化，允许将容器化的应用程序部署到集群中。为了使用这种新的部署模型，需要将应用程序和使用环境一起打包成容器。与过去的部署模型相比，容器化的应用程序更加灵活和可用，在新的部署模型中，应用程序被直接安装到特定的机器上，Kubernetes能够以更高效的方式在集群中实现容器的分发和调度运行。

 Kubernetes集群包括两种类型资源：

- [Master节点](http://docs.kubernetes.org.cn/306.html)：协调控制整个集群。
- [Nodes节点](http://docs.kubernetes.org.cn/304.html)：运行应用的工作节点。

集群架构图:

![](http://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

> Master 负责集群的管理。Master 协调集群中的所有行为/活动，例如应用的运行、修改、更新等。
>
> （Node）节点作为Kubernetes集群中的工作节点，可以是VM虚拟机、物理机。每个node上都有一个Kubelet，用于管理node节点与Kubernetes Master通信。每个Node节点上至少还要运行container runtime（比如docker或者rkt）。
>
> Kubernetes上部署应用程序时，会先通知master启动容器中的应用程序，master调度容器以在集群的节点上运行，node节点使用master公开的Kubernetes API与主节点进行通信。最终用户还可以直接使用Kubernetes API与集群进行交互。
>
> Kubernetes集群可以部署在物理机或虚拟机上。使用Kubernetes开发时，你可以采用[Minikube](http://docs.kubernetes.org.cn/94.html)。Minikube可以实现一种轻量级的Kubernetes集群，通过在本地计算机上创建虚拟机并部署只包含单个节点的简单集群。Minikube适用于Linux，MacOS和Windows系统。Minikube CLI提供集群管理的基本操作，包括 start、stop、status、 和delete。



**1.下载kubectl**

```bash
curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/v1.6.4/bin/linux/amd64/kubectl
chmod +x kubectl
```

**2. 安装minikube**

```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && \
  chmod +x minikube && \
  sudo mv minikube /usr/local/bin/
```

**3.启动minikube**

```bash
minikube start

#显示信息如下
There is a newer version of minikube available (v1.3.1).  Download it here:
https://github.com/kubernetes/minikube/releases/tag/v1.3.1
To disable this notification, add WantUpdateNotification: False to the json config file at /home/mrwhite/.minikube/config
(you may have to create the file config.json in this folder if you have no previous configuration)
Starting local Kubernetes cluster...


Kubernetes is available at https://192.168.99.100:8443.
Kubectl is now configured to use the cluster.
```

验证kubectl的配置

```bash
kubectl cluster-info
```

#### 使用minikube在k8s中运行应用

**1.创建node.js应用程序**

```bash
mkdir k8s
cd k8s
vim server.js
# 文本内容如下
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!');
};
var www = http.createServer(handleRequest);
www.listen(8080);
```

运行应用:

```bash
node server.js
```

在浏览器的窗口输入localhost:8080 可以查看到 "Hello world的消息"

**2.创建Docker容器镜像**

Dockerfile:

```dockerfile
FROM node:6.9.2
EXPOSE 8080
COPY server.js .
CMD node server.js
```

使用minikube 不是将Docker镜像push到registry，可以使用与Minikube VM相同的Docker主机构建镜像，以使镜像自动存在。为此，请确保使用Minikube Docker守护进程

```bash
eval $(minikube docker-env)
```

注意：如果不在使用Minikube主机时，可以通过运行eval $(minikube docker-env -u)来撤消此更改。

使用Minikube Docker守护进程build Docker镜像：

```
docker build -t hello-node:v1 .
```

构建过程中可能会出现docker客户端和服务端版本不一致的情况:

```bash
# 将版本更改为一致的
export DOCKER_API_VERSION=1.23
```

**3.创建Depolyment**

Deployment管理Pod的创建和扩展,[Pod](http://docs.kubernetes.org.cn/312.html)是一个或多个容器组合在一起得共享资源

使用kubectl run命令创建Deployment来管理Pod。Pod根据hello-node:v1Docker运行容器镜像：

```bash
kubectl run hello-node --image=hello-node:v1 --port=8080
```

查看Depolyment:

```bash
kubectl get deployments

#输出
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           3m

# 查看pod 
kubectl get pods

#输出
NAME                         READY     STATUS    RESTARTS   AGE
hello-node-714049816-ztzrb   1/1       Running   0          6m

# 查看群集events
kubectl get events

# 查看kubectl配置
kubectl config view
```

**问题:**

> ImagePullBackOff: "Back-off pulling image \"gcr.io/google_containers/pause-amd64:3.0 解决方法:
>
> 1 进入minikube ssh
>
> minikube ssh
>
> 2 拉取镜像
>
> docker pull registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0
>
> 3 打tag
>
> docker tag registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0
>
> 4 退出
>
> exit

#### 创建Service

默认情况，这Pod只能通过Kubernetes群集内部IP访问。要使hello-node容器从Kubernetes虚拟网络外部访问，须要使用Kubernetes [Service](http://docs.kubernetes.org.cn/703.html)暴露Pod。

将pod暴露到外部环境

```bash
kubectl expose deployment hello-node --type=LoadBalancer
```

查看创建的service

```bash
kubectl get services
# 输出
NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
hello-node   10.0.0.187   <pending>     8080:30501/TCP   9s
kubernetes   10.0.0.1     <none>        443/TCP          1h

```

通过--type=LoadBalancer flag来在群集外暴露Service，在支持负载均衡的云提供商上，将配置外部IP地址来访问Service。在Minikube上，该LoadBalancer type使服务可以通过minikube Service 命令访问。

```bash
minikube service hello-node
```

将打开浏览器，在本地IP地址为应用提供服务，显示“Hello World”的消息。

查看日志:

```bash
kubectl logs <POD-NAME>
```

更新service.js

```
response.end('Hello World Again!');
```

构建新的镜像

```bash
docker build -t hello-node:v2 .
```

deployment 更新镜像	

```bash
kubectl set image deployment/hello-node hello-node=hello-node:v2
```

再次查看消息

```bash
minikube service hello-node
```

#### 清理删除

删除在集群中创建的资源

```bash
kubectl delete service hello-node
kubectl delete deployment hello-node
```

```bash
# 停止minikube
minikube stop
```



