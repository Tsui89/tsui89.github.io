##k2-compose Manuals

####简介
k2-compose集成了docker-compose==1.7.1的基础功能，包括
	
	up、stop、start、restart、rm、logs、pull

优化或者增加了	

	inspect、ps、bash、show	
配置文件支持docker-compose原生、k2-compose定制化，这两种配置文件格式。

####特点
相对于docker-compose，另外还支持：

	1. 主机列表
	2. 容器列表
	3. 容器指定运行主机
	4. 容器依赖关系
	5. 容器健康检查
	6. 容器镜像的信息检测

####k2-compose文件示例
```yaml
version: "2"
hosts:
  as: 10.1.10.48:4243

project: k2-compose-test

services:

  busybox1:
    image: busybox:latest
    health_check:
      shell: echo "ok." && exit 0
      timeout: 10
    entrypoint: ["ping", "localhost"]

  busybox2:
    image: busybox:latest
    host: as
    health_check:
      shell: echo "ok." && exit 1
      timeout: 10
    entrypoint: ["ping", "localhost"]
    s_depends_on:
      - busybox1
```
上述例子中hosts、project、host、health_check、s_depends_on是k2-compose专有字段，

| 字段 | 描述 | 类型 | required |缺省默认值|
|---|---|----|---|---|
|hosts|主机列表|object|false|"local":"127.0.0.1:4243"
|project|项目名称；缺省默认|string|false|"k2-compose"
|services.service.host|容器运行主机名，必须在hosts里定义|string|false|local
|services.service.s\_depends\_on|依赖的容器列表|array|false|空|
|services.service.health\_check|健康检查命令属性|object|false|空（健康）|
|services.service.health\_check.shell|检查命令shell格式|string or string数组|false
|services.service.health\_check.timeout|检查命令超时时间|int(单位s)|false|10|


####操作示例
以下k2-compose.yml就是“k2-compose文件示例”的内容。

#####ps
这时候容器还没有部署，Service-Status是undeployed

```shell
root@minion1:~/k2-compose-0.0.4rc1/tests# k2-compose -f k2-compose.yml  ps
+----------+----------------+----------------+--------------+------------+-------+--------------+
| Service  | Host           | Service-Status | Image-Status | Depends-On | Ports | Network-Mode |
+----------+----------------+----------------+--------------+------------+-------+--------------+
| busybox1 | 127.0.0.1:4243 | undeployed     |              |            |       | default      |
+----------+----------------+----------------+--------------+------------+-------+--------------+
| busybox2 | localhost:4243 | undeployed     |              | - busybox1 |       | default      |
+----------+----------------+----------------+--------------+------------+-------+--------------+
```
