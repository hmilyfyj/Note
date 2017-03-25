title: 持续集成、部署方案
date: 2017-03-25 09:29:15
tags: [CI,CD,运维]
categories: 运维
---

<!-- more -->

## 阶段一
大学时部署、传递代码的工具是 Haozip、FTP


## 阶段二
刚工作时用 Git、脚本检测定时 pull （那时项目没有包依赖）

## 阶段三
现在项目用了`Composer`，那就不能简单的在线上构建了，容易因不可抗力出问题。

因此，制订了新的构建、部署流程。

## 用到的工具

`Gitlab`：存放源码
`Gitlab-Runner`: 用于构建代码
`Ansible`: 运维工具
`ansistrano`: 基于 `Ansible` 的部署工具

### 构建思路

代码推送到`Gitlab`后触发构建事件，部署在服务器的Gitlab-Runner（Docker 镜像）开始运转：安装依赖、执行单元测试。测试通过后部署到生产服务器上。


## 详细步骤

### Gitlab 控制整体流程

项目根部放入 `.gitlab-ci.yml` 文件：

````Ini
image: daocloud.io/hmilyfyj/php-fpm70:latest

variables:
  BASE_CONF_DIR: "/external_file"
  ENV_CONF_PATH: ${BASE_CONF_DIR}/config/web/${CI_PROJECT_PATH}/production.env
  ANSIBLE_DIR: ${BASE_CONF_DIR}/ansible/${CI_PROJECT_PATH}
  ANSIBLE_CONFIG: ${BASE_CONF_DIR}/ansible/common/ansible.cfg
  DEFAULT: "/external_file"

before_script:
  - pwd
  - ls -l
  - cat ~/.bashrc
  - source ~/.bashrc

### 代码构建
buid_script:
  only:
    - master
  stage: build
  script:
    - pwd
    - composer install
    - composer dump-autoload -o
  artifacts:
    untracked: true

### 测试
test_script:
  only:
    - master
  stage: test
  script:
    - phpunit
  artifacts:
    untracked: true

### 部署到测试环境

### 部署（构造生产环境、分发文件、重启 nginx） ansible 是一款部署工具
deploy_script:
  only:
    - master
  stage: deploy
  script:
    - rm .git -rf
    - cp ${ENV_CONF_PATH} .env
    - ansible-playbook -i ${ANSIBLE_DIR}/hosts ${ANSIBLE_DIR}/deploy.yml
````

#### ansistrano 部署代码

````Ini
---
- name: Deploy
  hosts: docker_machine
  vars:
    ansistrano_deploy_from: "/builds/huigou/admin_v3"
    ansistrano_deploy_to: "/usr/local/docker_share/home/wwwroot/huigou/admin_v3"
    ansistrano_keep_releases: 3
    # There seems to be an issue with rsync in vagrant
    ansistrano_deploy_via: rsync
  roles:
    - { role: carlosbuenosvinos.ansistrano-deploy }


````