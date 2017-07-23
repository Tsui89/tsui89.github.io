---
layout: post
comments: true
title:  "Mesos in Action (一) 初识"
date:   2017-07-23 15:01:19 +0800
categories: jekyll update
---

## Mesos 架构概念

传统上，数据中心内的物理机、虚拟机是典型的计算单元，由系统管理员来管理其日常运作。
Mesos通过引入一层抽象，提供一种像管理单台大服务器般的方法来管理整个数据中心。
而相对于hypervisor抽象cpu、内存、磁盘，之后以虚拟机形式提供，Mesos抽象资源之后，直接提供给应用。

##### 应用部署方式对比

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/mesos/app.jpg" />
</div>

##### Mesos 架构图

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/mesos/struct.jpg"/>
</div>

* Mesos 从底层系统抽象化CPU、内存、磁盘资源，将多个服务器展现成一个大机器
* Mesos slaves 以资源供给的形式宣告它有效的CPU、内存、磁盘资源
* Mesos framework 由两个主要组件组成：scheduler、executor
* 容器使用轻量级方法实现了进程间的资源隔离
