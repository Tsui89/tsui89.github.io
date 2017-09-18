---
layout: post
comments: true
title:  "Troubleshooting in DC/OS"
date:   2017-08-21 17:01:19 +0800
categories: jekyll update
---

部署方式: shell command


基础环境搭建

```shell
#!/bin/bash
yum install -y docker
cp daemon.json > /etc/docker/daemon.json
systemctl enable docker
systemctl start docker
systemctl stop firewalld && systemctl disable firewalld

groupadd docker
groupadd nogroup
yum install -y unzip
yum install -y ntp
yum install -y bind-utils
systemctl enable ntpd
systemctl start ntpd
timedatectl set-ntp true
sed -i s/SELINUX=enforcing/SELINUX=disabled/g /etc/selinux/config
```

cat daemon.json

```json
{
  "registry-mirrors" : [
      "https://registry.docker-cn.com"
    ],
  "storage-driver":"overlay",
  "hosts": [
      "unix:///var/run/docker.sock",
      "tcp://0.0.0.0:4243"
    ]
}
```

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

2. Failed to start Navstar: A distributed systems & network overlay orchestration engine

  参考  [http://alexjoh.blogspot.com/2017/04/dcos-troubleshooting.html](http://alexjoh.blogspot.com/2017/04/dcos-troubleshooting.html)

  在master上修改overlay配置
  vi /opt/mesosphere/etc/overlay/config/master.json

3. Ignoring kill task download.3f71d9f7-9aba-11e7-bbf1-92fccfe92fe6.55 because the executor 'download.3f71d9f7-9aba-11e7-bbf1-92fccfe92fe6.55' of framework 5843c95e-fd21-4866-8207-8780c517b6ab-0001 at executor(1)@192.168.131.3:33666 is terminating

  * rm -f /var/lib/mesos/slave/meta/slaves/latest
  * systemctl restart dcos-mesos-slave

4.  dcos-spartan-watchdog.service                                                                    loaded activating start-pre start DNS Forwarder (Spartan) W

  *  service docker restart
