---
layout: post
comments: true
title:  "Troubleshooting"
date:   2017-08-21 17:01:19 +0800
categories: jekyll update
---

部署方式: shell command


基础环境
1. 关闭SELINUX，修改/etc/selinux/config
- systemctl stop firewalld && systemctl disable firewalld
- docker storage 不能使用devicemapper loop-lvm, \-\-storage-driver=overlay
- groupadd docker
- groupadd nogroup
- yum install unzip,docker
