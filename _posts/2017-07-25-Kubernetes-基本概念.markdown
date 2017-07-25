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
而后由Label Selector来定义所选择的对象。
