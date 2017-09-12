---
layout: post
comments: true
title:  "Troubleshooting in DC/OS"
date:   2017-08-21 17:01:19 +0800
categories: jekyll update
---

部署方式: shell command


基础环境
1. 关闭SELINUX，修改/etc/selinux/config
2. systemctl stop firewalld && systemctl disable firewalld
3. docker storage 不能使用devicemapper loop-lvm, \-\-storage-driver=overlay
4. groupadd docker
5. groupadd nogroup
6. yum install unzip,docker
7. systemctl enable docker



Troubleshooting:

1. bash dcos_generate_config.sh --genconf Failed to open config file at genconf/config.yaml: [Errno 13] Permission denied

```
[root@bootstrap bootstrap]# sudo !!
sudo bash dcos_generate_config.sh --genconf
====> EXECUTING CONFIGURATION GENERATION
Failed to open config file at genconf/config.yaml: [Errno 13] Permission denied: 'genconf/config.yaml'. See the DC/OS documentation to learn how to create a config file. You can also use the GUI web installer (--web), which provides a guided configuration and installation for simple deployments.
[root@bootstrap bootstrap]# systemctl stop firewalld
[root@bootstrap bootstrap]# systemctl disable firewalld
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
Removed symlink /etc/systemd/system/basic.target.wants/firewalld.service.
[root@bootstrap bootstrap]# sudo bash dcos_generate_config.sh --genconf
/usr/bin/docker-current: Error response from daemon: driver failed programming external connectivity on endpoint dcos-genconf.e38ab2aa282077c8eb-4d92536e7381176206.tar (1dd38968a8638d1a7ad137a50b8bf7a0049c75343d77d907ce713095231ed732): iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 9000 -j DNAT --to-destination 172.17.0.2:9000 ! -i docker0: iptables: No chain/target/match by that name.
 (exit status 1).
[root@bootstrap bootstrap]# vi /etc/selinux/config
[root@bootstrap bootstrap]# reboot
```

Failed to start Navstar: A distributed systems & network overlay orchestration engine
http://alexjoh.blogspot.com/2017/04/dcos-troubleshooting.html

在master上修改overlay配置
vi /opt/mesosphere/etc/overlay/config/master.json
