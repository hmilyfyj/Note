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

# 一、生产环境

`PHP`(5.6 + 7.0)、`Laravel`、`Memcached`（SASL）、`Redis`、`Mysql`、`Nginx`、`Centos`、`Gitlab`（CI、CD）、`Ansible`（快速部署）

上次服务器崩溃时，以上东西都要在新服务器上从头开始安装，也不熟练安装了好久...
痛定思痛后，决定 **上 Docker** !

## 升级
````
yum update -y
````

## 安全配置

### 修改 ssh 端口

cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/
https://sebastianblade.com/how-to-modify-ssh-port-in-centos7/
https://linux.cn/article-8098-1.html
https://www.jevin.org/centos7-change-ssh-port/

踩坑：realod 会重置 iptables，直接影响容器运行。重启 docker 或机器
[issues](https://github.com/moby/moby/issues/16137)

## 安装 Docker

````shell
//get.docker.com
curl -sSL https://get.daocloud.io/docker | sh 
systemctl start docker
````
[相关资料](https://yeasy.gitbooks.io/docker_practice/content/install/centos.html#使用脚本自动安装)

配置 Docker

``` shell
#开机启动
systemctl enable docker

#登陆daocloud 私有云
docker login daocloud.io
```

安装 compose

``` shell	
curl -L https://get.daocloud.io/docker/compose/releases/download/1.14.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

[参考资料](https://get.daocloud.io/#install-compose)

pull 必要的镜像
````
docker pull daocloud.io/hmilyfyj/php-fpm56:latest
docker pull daocloud.io/hmilyfyj/php-fpm70:latest
docker pull nginx
````

3.安装 gitlab runner 相关
````
# For Debian/Ubuntu
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash

# For RHEL/CentOS
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash


# For Debian/Ubuntu
sudo apt-get install gitlab-ci-multi-runner

# For RHEL/CentOS
sudo yum install gitlab-ci-multi-runner

gitlab-runner register
````
https://docs.gitlab.com/runner/register/index.html
https://docs.gitlab.com/runner/install/linux-repository.html

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

修改 gitlab-runner config

//添加端口 28941 22 80 443 9000 3002


### 布置部署环境

### 布置生产环境