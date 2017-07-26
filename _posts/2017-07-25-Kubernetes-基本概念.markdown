---
layout: post
comments: true
title:  "Kubernetes 基本概念"
date:   2017-07-25 01:01:19 +0800
categories: jekyll update
---

#### Node

Node: Kubernetes集群中的工作主机，其上运行的服务包括：Kubelet、kube-proxy、docker-daemon。

Node信息： kubectl describe node <node-name>
* Node地址
* Node运行状态：Pending、Running、Terminated
* Node Condition：描述Running状态的Node运行条件，Ready表示处于健康状态
* Node可用系统资源
* 其他信息

#### Pod

Pod: 最小操作单元，运行与Node之上。

一个Pod内的容器共享资源：
* PID命名空间
* 网络命名空间
* UTS命名空间
* IPC命名空间：Pod内多个容器能通过System IPC或Posix消息队列进行通信
* Volumes

Pod状态：
* Pending
* Running
* Succeeded
* Failed

#### Label

Label： Kubernetes系统内的一个核心概念，以健值对的形式标记在各种对象上。如Pod、Node、Service、RC。
它定义的是Kubernetes对象的元数据，而后由Label Selector来定义所选择的对象。

#### Replication Controller

RC: 用于定义Pod副本的数量，在Master主机内，进程Replication Controller通过RC的定义
来控制Pod的创建、起停、销毁。删除RC并不会影响已创建的Pod，可以通过设置replicas=0来删除Pod。

#### Service

Service：一组提供相同服务的Pod对内外的统一访问接口。
* Pod ip是由docker0网络创建的，Service cluster ip是由kubernetes创建的。
* Pod ip随Pod的创建、销毁而发生变化，cluster ip只要Service不删除，就不会变。
* cluster ip可以供内部所有Pod进行连接。
* cluster ip也可以提供外部服务，方式：
  * NodePort
  * LoadBalancer

#### Volumes

Volumes类型：
* EmptyDir
  * 临时空间
  * 多容器共享目录
* hostPath
  * 永久保存
  * 获取主机数据信息
* gcePersistentDisk
* awsElasticBlockStore
* nfs
* iscsi
* glusterfs
* rbd
* gitRepo
* secret
* persistentVolumeClaim

#### Namespace

Namespace：通过将集群内部对象分发到不同的Namespace上，形成逻辑上分组。即多租户管理。

#### Annotation

Annotation：用户定义的附加信息，如：
* build信息，release信息
* 日志库、监控库信息
* 团队信息
