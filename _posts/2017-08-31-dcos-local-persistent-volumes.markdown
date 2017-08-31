MESOS 本地持久化存储类型：

1. root，最基本的存储资源，默认是mesos agent工作路径的存储资源。

    * 应用共享式使用。

    * 在创建应用的同时，会在/var/lib/mesos/slave/volumes目录下创建应用的存储目录。

    * agent上报root类型存储2G。
    ```
    [
     {
       "name" : "disk",
       "type" : "SCALAR",
       "scalar" : { "value" : 2048 }
     }
    ]
    ```

* path,作为附加存储资源，可以将整块磁盘，通过目录存储的方式进行划分。用于日志存储、备份，用于无性能要求的存储。

    * 应用共享式使用。

    * 在创建应用的同时，会在path目录下创建应用的存储目录。

    * 如 mount /dev/sdb /mnt && mkdir /mnt/data, /dev/sdb 是10G，agent上报目录存储资源/mnt/data，使用2G大小。
    ```
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

* mount,作为附加存储资源，只能整块使用，不能像path类存储一样分割使用。用于database、write-ahead-log(WAL预写式日志)，用于有性能要求的存储。

    * 应用独占式使用。

    * 在创建应用时，应用使用整个磁盘资源，允许有预存在的数据资源。

    * 如 mount /dev/sdb /mnt/data, agent上报存储磁盘资源/mnt/data,使用大小2G。
    ```
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
