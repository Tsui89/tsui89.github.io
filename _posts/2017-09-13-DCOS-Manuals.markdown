---
title:  "DC/OS 实践经验"
---

如何使用一个PaaS集群，其实就是如何使用它的网络、存储，这两点关系着应用的部署。
DC/OS集群中，网络分为Host、Bridge、Virtual Network。
应用的本地存储是通过挂载的形式来实现的，挂载分为Persistent Volume和Host Volume。
而Persistent Volume又分为root、path、mount类型，Host Volume相当于docker volume mount。详细介绍可见[DC/OS-local-persistent-volumes](https://tsui89.github.io/2017/08/31/DCOS-local-persistent-volumes.html).

##### 网络

普通应用网络模式选择Host、Bridge、Virtual Network都可以，外部访问可以设置端口映射。

集群应用、多个应用之间需要进行通信的，建议选择Virtual Network，如果需要外部访问服务，可以设置端口映射。

DC/OS集群使用mesos-dns进行服务解析，应用服务名称定义：
* 应用名.应用组名.marathon.mesos 获取的是应用容器内部ip，如果使用Host模式，就是宿主机ip
* 应用名.应用组名.marathon.slave.mesos 获取的是应用的宿主机ip

##### 存储

DC/OS集群本地存储

Persistent Volume，应用在一个节点上创建成功，之后的创建、重启都只在这一个节点上。
* root，一般用的是系统盘存储，不建议使用
* path，存储路径是由DC/OS管理员设置的，安全可靠，建议使用
* mount，特殊应用，对存储有性能要求的应用使用

Host Volume，将本地的目录／文件挂载到容器。这种存储类型下，容器可能会漂移到其他节点，所以有强烈部署节点限制的应用可以使用，或者搭配root、path、mount使用。

存储不光用于应用数据的存储，还用于应用配置文件的挂载。
对于一个需要外挂配置文件来启动的应用，部署流程是这样的：
1. 将配置文件上传到文件服务器，目录需要打成tar.gz包。
2. 应用的配置页面，设置artifact，下载文件/tar包的url
3. 存储配置页面，存储类型选Host Volume,将文件/目录名挂载到容器内路径

#### 示例 bind9外部挂载named.conf、db目录

##### 部署artifact-store


服务配置

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg1-service.png" />
</div>

设置部署主机

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg1-constraints.png" />
</div>

网络配置，内部使用artifact-store.microservices.marathon.mesos访问

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg1-network.png" />
</div>

挂载本地路径，指的目的主机之后，所有的数据文件都放在这个主机路径下

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg1-volume.png" />
</div>
