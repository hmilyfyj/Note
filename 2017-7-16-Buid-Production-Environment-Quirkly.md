---
title: 快速构建生产环境（1）
date:  2017-07-16 9:34:33
tags: [Linux,PHP,挖坑,运维]
categories: Linux
grammar_cjkRuby: true
---

经历了`阿里云`两次崩溃，从 0 开始搭建生产环境；加上昨日`Azure`断电导致`Gitlab CE`无法访问后。
一方面知道要给云计算宽容，另一方面也坚定了要有比较好的容灾机制，以及灾后迅速重建的手段。

<!-- more -->

---

[持续集成、部署方案](https://b.fengbl.cn/2017/03/25/2017-3-25-Plan-Of-CI-CD/)

### 一、生产环境

`PHP`(5.6 + 7.0)、`Laravel`、`Memcached`（SASL）、`Redis`、`Mysql`、`Nginx`、`Centos`、`Gitlab`（CI、CD）、`Ansible`（快速部署）

上次服务器崩溃时，以上东西都要在新服务器上从头开始安装，也不熟练安装了好久...
痛定思痛后，决定 **上 Docker** !


1.安装 Docker
````shell
//get.docker.com
curl -sSL https://get.daocloud.io/docker | sh 
systemctl start docker
````

https://yeasy.gitbooks.io/docker_practice/content/install/centos.html#使用脚本自动安装

登陆 daocloud.io 或者其他平台

``` shell
docker login daocloud.io
```

pull 必要的镜像


2.pull 镜像（nginx、php-fpm70、56）
3.2.安全加固
3.安装 gitlab runner 相关
修改 gitlab 配置（pull 政策）


修改端口

````
systemctl enable firewalld
systemctl start firewalld
systemctl status firewalld
firewall-cmd --set-default-zone=public
firewall-cmd --zone=public --add-interface=eth0
firewall-cmd --zone=public --add-interface=eth1
systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=22/tcp
systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=28941/tcp
systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=80/tcp
systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=443/tcp
systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=9000/tcp
systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=3002/tcp
systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=8901/tcp
systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=32809/tcp
firewall-cmd --reload
firewall-cmd --permanent --list-port
firewall-cmd --zone=public --list-all

systemctl start firewalld \
& firewall-cmd --set-default-zone=public \
& firewall-cmd --zone=public --add-interface=eth0 \
& firewall-cmd --zone=public --add-interface=eth1 \
& systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=22/tcp \
& systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=28941/tcp \
& systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=80/tcp \
& systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=443/tcp \ 
& systemctl start firewalld & firewall-cmd --permanent --zone=public --add-port=9000/tcp \
& firewall-cmd --reload \
& firewall-cmd --permanent --list-port \
& firewall-cmd --zone=public --list-all
````

踩坑：realod 会重置 iptables，直接影响容器运行。重启 docker 或机器
[issues](https://github.com/moby/moby/issues/16137)

//添加端口 28941 22 80 443 9000 3002

//