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
