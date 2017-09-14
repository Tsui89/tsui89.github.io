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
对于需要指定特殊存储位置的应用(比如mysql)，Persistent Volume需要配合Host Volume使用。
先将Persistent Volume挂载到容器的目录／文件（容器根目录相对路径），
然后将目录／文件通过Host Volume的形式挂载到容器数据存储绝对路径（比如/var/lib/mysql）。
* root，一般用的是系统盘存储，不建议使用
* path，存储路径是由DC/OS管理员设置的，安全可靠，建议使用
* mount，特殊应用，对存储有性能要求的应用使用

Host Volume，将本地的目录／文件挂载到容器。这种存储类型下，容器可能会漂移到其他节点，所以有强烈部署节点限制的应用可以使用，或者搭配root、path、mount使用。
* Host Volume 不光可以挂载主机本地路径，还可以挂载artifact下载的文件/目录

存储不光用于应用数据的存储，还用于应用配置文件的挂载。
对于一个需要外挂配置文件来启动的应用，部署流程是这样的：
1. 将配置文件上传到文件服务器，目录需要打成tar.gz包。
2. 应用的配置页面，设置artifact，下载文件/tar包的url
3. 存储配置页面，存储类型选Host Volume,将文件/目录名挂载到容器内路径


#### 示例一 部署artifact-store，Virtual Network + Host Volume


服务配置

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg1-service.png" />
</div>

设置部署主机

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg1-constraints.png" />
</div>

网络配置，内部使用域名artifact-store.microservices.marathon.mesos访问

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg1-network.png" />
</div>

挂载本地路径，使用的宿主机绝对路径，所有的artifact文件都需要上传到这个宿主机路径下

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg1-volume.png" />
</div>

然后run service


#### 示例二 部署bind9 应用，Host Network + Host Volume

首先将bind9使用的配置文件named.conf bind9.db.tar.gz(artifact下载之后默认会解压，我们真正使用也是解压之后的db目录)上传到/artifact-store目录
```
[root@dcos-cloud1 ~]# ls /artifact-store/
bind9.db.tar.gz  named.conf
```

服务配置

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg2-service.png" />
</div>

设置部署主机,设置artifact下载url

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg2-artifact.png" />
</div>

网络配置

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg2-network.png" />
</div>

挂载本地路径，这个地方使用的是相对路径（应用容器的存储根目录），artifact下载之后就放在应用容器根目录下，所以直接挂载named.conf、db

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg2-volume.png" />
</div>

然后run service

#### 示例三 部署 test-root 应用，Host Network + Root Volume + Host Volume

服务配置

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg3-service.png" />
</div>

网络配置

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg3-network.png" />
</div>

挂载Persistent Volume默认类型root到容器跟目录下的mydata，然后将mydata挂载到容器/tmp目录下

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg3-volume.png" />
</div>


#### 示例四 部署 influxdb 应用，Host Network + Path Volume + Host Volume

DC/OS 默认的Persistent Volume类型是root，修改类型，需要打开右上方的JSON EDITER。
修改persistent.type = "path"
```JSON
"volumes": [
      {
        "persistent": {
          "type": "path",
          "size": 1000000,
          "constraints": []
        },
        "mode": "RW",
        "containerPath": "influxdb-data"
      },
      {
        "containerPath": "/var/lib/influxdb",
        "hostPath": "influxdb-data",
        "mode": "RW"
      }
    ],
```

服务配置

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg4-service.png" />
</div>

网络配置

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg4-network.png" />
</div>

挂载path volume到/var/lib/influxdb。
首先在页面上选择Persistent Volume，配置大小，挂载容器存储根目录下的influxdb-data，再创建Host Volume，
将容器存储根目录下的influxdb-data挂载到容器内数据存储路径/var/lib/influxdb。
然后打开JSON EDITER,修改persistent{ "type": "path"}

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg4-volume.png" />
</div>

环境变量配置

<div>
	<img width="100%" src="https://tsui89.github.io/static/posts/dcos/eg4-env.png" />
</div>

然后run service
