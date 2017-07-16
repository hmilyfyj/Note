---
title: 快速构建生产环境（1）
date:  2017-07-16 9:34:33
tags: [Linux,PHP,挖坑,运维]
categories: Linux
grammar_cjkRuby: true
---

经历了`阿里云`崩溃两次，从 0 开始搭建生产环境；加上昨日`Azure`断电导致`Gitlab CE`无法访问后。
一方面知道要给云计算宽容，另一方面也坚定了要有比较好的容灾机制，以及灾后迅速重建的手段。

<!-- more -->

---

[持续集成、部署方案](https://b.fengbl.cn/2017/03/25/2017-3-25-Plan-Of-CI-CD/)

### 一、生产环境

`PHP`(5.6 + 7.0)、`Laravel`、`Memcached`（SASL）、`Redis`、`Mysql`、`Nginx`、`Centos`、`Gitlab`（CI、CD）、`Ansible`（快速部署）

上次服务器崩溃时，以上东西都要在新服务器上从头开始安装，也不熟练安装了好久...
痛定思痛后，决定 **上 Docker** !