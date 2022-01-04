---
title: 开启阿里云 ECS 的 NAT 的功能
date: 2020-10-10 23:31:38
tags: Aliyun,Centos
categories: Aliyun
img: /media/16413041801999.jpg
---

## Centos

现在后台开启转发

```
# 开启firewalld防火墙，默认是关闭的。
$ systemctl enable firewalld
$ systemctl start firewalld

# 网卡默认是在public的zone内，也是默认zone。永久添加源地址转换功能
$ firewall-cmd --add-masquerade --permanent
$ firewall-cmd --reload
 
# 添加网卡的ip转发功能，添加如下配置到文件最后
$ vim /etc/sysctl.conf
----------------------------------------------------------------
net.ipv4.ip_forward=1
----------------------------------------------------------------
  
# 重载网络配置生效
$ sysctl -p
```

注意，如果系统中有 docker 容器，建议重启 docker 或主机，因为 docker 也用到了 iptables，所以 iptables 可能会被影响。

## 后台

在 vpc 中开启转发。

![](/media/16023442184292.jpg)