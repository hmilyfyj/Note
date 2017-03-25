title: 持续集成、部署方案
date: 2017-03-25 09:29:15
tags: [CI,CD,运维]
categories: 运维
---

## 阶段一
大学时部署、传递代码的工具是 Haozip、FTP

<!-- more -->

---

## 阶段二
刚工作时用 Git、脚本检测定时 pull （那时项目没有包依赖）

## 阶段三
现在项目用了`Composer`，那就不能简单的在线上构建了，容易因不可抗力出问题。

因此，制订了新的构建、部署流程。

### 用到的工具

`Gitlab`：存放源码
`Gitlab-Runner`: 用于构建代码
`Ansible`: 运维工具
`ansistrano`: 基于 `Ansible` 的部署工具

## 流程