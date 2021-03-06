title: 一台设备管理多个 Git 帐号的仓库
date: 2016-04-19 16:56
tags: [Git]
categories: Git
---

Git、Coding 等网站都不允许同一个 ssh 公钥 绑定多个帐号。于是做些配置来“曲线救国”。

<!-- more -->

---


## 配置

### 生成密钥

生成多个不同名称的密钥，如：

```
ssh-keygen -t rsa -C "mywork@email.com" // fengit_id_rsa
ssh-keygen -t rsa -C "mywork@email.com" // hmilyfyj_id_rsa
```

### 配置文件

在 `.ssh` 目录下配置 `config` 文件：

```
Host fengit
HostName github.com
User git
IdentityFile ～/.ssh/id_rsa_fengit

Host hmilyfyj
HostName github.com
User git
IdentityFile ～/.ssh/id_rsa_hmilyfyj
```

## 测试

```
ssh -T fengit
Hi fengit! You've successfully authenticated, but GitHub does not provide shell access.
```

## 连接仓库

新方法：

	git clone fengit:githubname/repository.git

原来的连接方法：

	git clone git@github.com:githubname/repository.git