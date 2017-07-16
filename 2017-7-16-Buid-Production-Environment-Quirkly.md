---
title: 快速构建生产环境（1）
date:  2017-07-16 9:34:33
tags: [Linux,PHP,挖坑,运维]
categories: Linux
grammar_cjkRuby: true
---

经历过了`阿里云`两次崩溃以及昨日`Azure`断电导致`Gitlab CE`无法访问：
一方面知道要给云计算宽容，另一方面也坚定了要有比较好的容灾机制，以及灾后迅速重建的手段。

<!-- more -->

---

# 一、生产环境

`PHP`(5.6 + 7.0)、`Laravel`、`Memcached`（SASL）、`Redis`、`Mysql`、`Nginx`、`Centos`、`Gitlab`（CI、CD）、`Ansible`（快速部署）

上次服务器崩溃时，以上东西都要在新服务器上从头开始安装，也不熟练安装了好久...
痛定思痛后，决定 **上 Docker** ! 争取在服务器挂掉的情况下能在最多半小时内恢复服务！


之前总结了 [持续集成、部署方案](https://b.fengbl.cn/2017/03/25/2017-3-25-Plan-Of-CI-CD/) 的笔记，省了很多麻烦，但目前两台服务器业务不同，有点乱，真的出现崩溃绝对不会太过轻松高效的部署，本篇笔记正是为了达成更加高效的目标而撰写。

从 阿里云购买了按量付费的 `ECS` 练手，一小时 0.6 元，便宜又好用，美滋滋~  现整理安装新机后的部署步骤，方便整理成一键部署脚本。

## 升级
````
yum update -y
````

## 安全（救急时跳过）

### 修改 ssh 端口
````shell
cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/
````
https://sebastianblade.com/how-to-modify-ssh-port-in-centos7/
https://linux.cn/article-8098-1.html
https://www.jevin.org/centos7-change-ssh-port/

踩坑：realod 会重置 iptables，直接影响容器运行。需要重启 docker 或机器
[issues](https://github.com/moby/moby/issues/16137)

修改 firewalld

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
````

//添加端口 28941 22 80 443 9000 3002

## 安装 Docker

### 安装
````shell
#get.docker.com 太慢
curl -sSL https://get.daocloud.io/docker | sh 
````
[相关资料](https://yeasy.gitbooks.io/docker_practice/content/install/centos.html#使用脚本自动安装)

### 配置 Docker

``` shell
#开机启动
systemctl enable docker
systemctl start docker

#登陆daocloud 私有云
docker login daocloud.io
```

### 安装 compose

``` shell	
curl -L https://get.daocloud.io/docker/compose/releases/download/1.14.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

[参考资料](https://get.daocloud.io/#install-compose)

### pull 必要的镜像
````shell
docker pull daocloud.io/hmilyfyj/php-fpm56:latest
docker pull daocloud.io/hmilyfyj/php-fpm70:latest
docker pull nginx
````

## 配置部署环境


### 安装 gitlab runner 相关
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

修改 gitlab 配置（pull 政策），修改文件 `/etc/gitlab-runner/config.toml` :

主要配置 `extra_hosts` 、`pull_policy`、`volumes` 参数

````xml
concurrent = 1
check_interval = 0

[[runners]]
  name = "test php"
  url = "https://gitlab.com/ci"
  token = "9b9293a2eda9da1caf8fd80e916bef"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "daocloud.io/hmilyfyj/php-fpm70"
    privileged = false
    disable_cache = false
    volumes = ["/cache", "/var/build_and_deploy:/external_file", "/root/.acme.sh:/var/sslcert"]
    pull_policy = "if-not-present"
    extra_hosts = ["gitlab.com:52.167.219.168"]
  [runners.cache]

````

### 布置部署环境

### 布置生产环境


ssh-copy-id user@server
ssh-copy-id -i ~/.ssh/id_rsa.pub user@server


http://guide.daocloud.io/dcs/docker-9152677.html