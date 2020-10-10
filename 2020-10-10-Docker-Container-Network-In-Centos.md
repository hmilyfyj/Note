---
title: Docker 容器无法访问网络的问题
date: 2020-10-10 23:07:28
tags: Docker,Centos
categories: Docker
---

最近把系统切换到了 Aliyun Linux，该系统是基于 Centos 做的改进。

安装生产环境的流程一切正常，毕竟大部分软件是基于 docker 运行。

但是在启动 docker 容器后遇到了问题：容器内无法访问网络，无论 ping、curl 还是wget，都执行失败 。

## 排查

### 排查 firewalld 故障
已排除。

不论添加信任的 interfaces、port，还是关闭 firewalld 服务，均无效果。

### 排查 docker 相关问题

[官方文档](https://docs.docker.com/engine/install/linux-postinstall/#ip-forwarding-problems)中有提到，当 systemd 版本不低于 219 时，可能存在网络问题。通过以下方式验证版本：

```
[root@iZbp1actuhd151iwb8g872Z ~]# systemctl --version
systemd 219
+PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 -SECCOMP +BLKID +ELFUTILS +KMOD +IDN
[root@iZbp1actuhd151iwb8g872Z ~]# 
```

这个原因的可能性较大，因为初始化系统时，我执行了 `yum update`，所以 systemd 更新到了最新版。通过参考文档的提示进行调整后，也验证了猜想，调整方法见下文。

## 解决

修改该目录（/etc/systemd/network）下的配置文件。这个目录我是在 sf 找到的，官方文档里不是这个目录，如果通过系统来确定目录的方法待查询。

```
[Network]
...
IPForward=kernel
# OR
IPForward=true
...
```

修改完成后，重启网络或者重启机器：`systemctl restart systemd-networkd`

[Containers cannot access internet (outbound) #36151](https://github.com/moby/moby/issues/36151#issuecomment-541096349)
[My docker container has no internet](https://stackoverflow.com/questions/20430371/my-docker-container-has-no-internet/60074328#60074328)
[ip-forwarding-problems](https://docs.docker.com/engine/install/linux-postinstall/#ip-forwarding-problems)