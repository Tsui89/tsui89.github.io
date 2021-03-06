---
title:  "DC/OS Local Persistent Volumes"
---

### MESOS 本地持久化存储类型：

1. root，最基本的存储资源，默认是mesos agent工作路径的存储资源。

    * 应用共享式使用。

    * 在创建应用的同时，会在/var/lib/mesos/slave/volumes目录下创建应用的存储目录。

    * agent上报root类型存储2G。

    ```json
        [
          {
            "name" : "disk",
            "type" : "SCALAR",
            "scalar" : { "value" : 2048 }
          }
        ]
    ```

2. path,作为附加存储资源，可以将整块磁盘，通过目录存储的方式进行划分。用于日志存储、备份，用于无性能要求的存储。

    * 应用共享式使用。

    * 在创建应用的同时，会在path目录下创建应用的存储目录。

    * 如 mount /dev/sdb /mnt && mkdir /mnt/data, /dev/sdb 是10G，agent上报目录存储资源/mnt/data，使用2G大小。

    ```json
        [
          {
            "name" : "disk",
            "type" : "SCALAR",
            "scalar" : { "value" : 2048 },
            "disk" : {
              "source" : {
                "type" : "PATH",
                "path" : { "root" : "/mnt/data" }
              }
            }
          }
        ]
    ```

3. mount,作为附加存储资源，只能整块使用，不能像path类存储一样分割使用。用于database、write-ahead-log(WAL预写式日志)，用于有性能要求的存储。

    * 应用独占式使用。

    * 在创建应用时，应用使用整个磁盘资源，允许有预存在的数据文件/目录，但是当mesos销毁应用时，会删除所有数据

    * 如 mount /dev/sdb /mnt/data, agent上报存储磁盘资源/mnt/data,使用大小2G。

    ```json
        [
          {
            "name" : "disk",
            "type" : "SCALAR",
            "scalar" : { "value" : 2048 },
            "disk" : {
              "source" : {
                "type" : "MOUNT",
                "mount" : { "root" : "/mnt/data" }
              }
            }
          }
        ]
    ```

### Marathon docker应用使用local persistent volume示例

Marathon对于这几种Persistent Volume的使用区分，主要是通过persistent.type来控制的。

操作流程：

    1. marathon创建应用
    2. 查看应用mount信息
    3. 创建tmp.txt
    4. 重启应用，查看tmp.txt
    5. 查看应用的本地存储目录

#### root

创建marathon 应用
```json
{
  "id": "/cwc/test/test-root",
  "instances": 1,
  "portDefinitions": [],
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "busybox"
    },
    "volumes": [
      {
        "persistent": {
          "size": 100
        },
        "mode": "RW",
        "containerPath": "mydata"
      },
      {
        "containerPath": "/mydata",
        "hostPath": "mydata",
        "mode": "RW"
      }
    ]
  },
  "cpus": 0.1,
  "mem": 128,
  "requirePorts": false,
  "cmd": "tail -f /dev/null",
  "residency": {
    "relaunchEscalationTimeoutSeconds": 10,
    "taskLostBehavior": "WAIT_FOREVER"
  }
}
```

这时候marathon随机选取了一个主机去部署应用，以后与这个应用相关的所有操作都只在这个主机上运行。
可以看到
1. 应用容器内存储目录 /mydata

2. 主机存储目录/var/lib/mesos/slave/volumes/roles/slave_public/cwc_test_test-root#mydata#743b88a7-8e17-11e7-b490-b6a91729f427

3. 重启应用数据不变

4. 重启agent，应用自动恢复，数据不变

5. 应用销毁，数据消失

#### path

marathon 应用

```json
{
  "id": "/cwc/test/test-path",
  "instances": 1,
  "portDefinitions": [],
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "busybox",
      "network": "HOST"
    },
    "volumes": [
      {
        "persistent": {
          "size": 100,
          "type": "path"
        },
        "mode": "RW",
        "containerPath": "mydata"
      },
      {
        "containerPath": "/mydata",
        "hostPath": "mydata",
        "mode": "RW"
      }
    ]
  },
  "cpus": 0.1,
  "mem": 128,
  "requirePorts": false,
  "cmd": "tail -f /dev/null",
  "residency": {
    "relaunchEscalationTimeoutSeconds": 10,
    "taskLostBehavior": "WAIT_FOREVER"
  }
}
```

这时候marathon随机选取了一个主机去部署应用，以后与这个应用相关的所有操作都只在这个主机上运行。
可以看到
1. 应用容器内存储目录 /mydata

2. 主机存储目录/mnt/data/volumes/roles/slave_public/cwc_test_test-path#mydata#8f2d2ba9-8e1c-11e7-b490-b6a91729f427/

3. 重启应用数据不变

4. agent重启，数据不变

5. 应用销毁，数据消失

6. agent 重新resize path resource，应用启动失败

#### mount

marathon 应用

```json
{
  "id": "/cwc/test/test-mount",
  "instances": 1,
  "portDefinitions": [],
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "busybox"
    },
    "volumes": [
      {
        "persistent": {
          "size": 100,
          "type": "mount"
        },
        "mode": "RW",
        "containerPath": "mydata"
      },
      {
        "containerPath": "/mydata",
        "hostPath": "mydata",
        "mode": "RW"
      }
    ]
  },
  "cpus": 0.1,
  "mem": 128,
  "requirePorts": false,
  "cmd": "tail -f /dev/null",
  "constraints": [
    [
      "hostname",
      "LIKE",
      "192.168.131.4"
    ]
  ],
  "residency": {
    "relaunchEscalationTimeoutSeconds": 10,
    "taskLostBehavior": "WAIT_FOREVER"
  }
}
```
1. 应用容器内存储目录 /mydata

2. 主机存储目录/dcos/volume0/

3. 重启应用数据存在

4. agent重启，数据存在

5. 应用销毁，数据消失

6. agent 重新resize mount resource，应用启动失败

### Marathon 使用Host Volume

```json
{
    "type": "DOCKER",
    "volumes": [
        {
            "mode": "RO",
            "container_path": "/etc/localtime",
            "host_path": "/etc/localtime"
        },
        {
            "mode": "RW",
            "container_path": "/tmp",
            "host_path": "/tmp"
        }
    ],
    "docker": {
        "image": "busybox",
        "network": "HOST",
        "privileged": false,
        "parameters": [
            {
                "key": "label",
                "value": "MESOS_TASK_ID=cwc_test-local-path.75db9606-97d6-11e7-aaa2-36ce7409b167"
            }
        ],
        "force_pull_image": false
    }
}
```

#### 资料检索

[mesos multiple-disk](http://mesos.apache.org/documentation/latest/multiple-disk)

[dcos persistent volume](https://dcos.io/docs/1.9/storage/persistent-volume/)

[marathon persistent volumes](https://mesosphere.github.io/marathon/docs/persistent-volumes.html)
