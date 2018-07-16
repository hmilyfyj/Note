---
title: 2018-7-13-PHP-Guide 
date: 2018-07-13 16:56:49
tags: PHP
categories: PHP
---


<!-- more -->

---

## 网络环境

公司网络已配置科/学/上/网，可访问 谷歌、Stack Overflow 查询资料。

*   [https://www.google.com](https://www.google.com/_/chrome/newtab?ie=UTF-8)

*   <https://stackoverflow.com>

## 开发环境

PHP 7.2.*

Mysql 5.6

搭建环境工具：可根据喜好选择 Docker 、Xampp 等。

## 开发工具

PHPStorm

激活方法：<http://idea.lanyus.com>

# 使用框架

*   Laravel <https://laravel-china.org/docs/laravel/5.6>

*   Lumen（与 Laravel 语法基本一致，专用于写 Api） <https://lumen.laravel-china.org/>

## PHPStorm 插件

代码规范：Code Sniffer，规范选择 PSR2

# 编码规范

### 参考文档

PSR-2：<https://laravel-china.org/docs/psr/psr-2-coding-style-guide/1606>

其他：<http://laravel-china.github.io/php-the-right-way/>

Tab 键：4 个空格，而不是制表符。

# 数据库设计规范

### 参考文档

<https://github.com/Rongx/teamCode/blob/master/mysql.md>

<https://github.com/zhishutech/mysql-sql-standard/blob/master/SUMMARY.md>

<https://blog.csdn.net/u012966918/article/details/52161519>

数据库设计工具：

*   使用 Navicat

*   使用 Laravel 自带的建表流程。

*   手写 SQL

# 接口设计规范

<http://jsonapi.org.cn>

<https://segmentfault.com/a/1190000008997508>

<https://www.jianshu.com/p/f15e4ebb5a2f>

# 什时候用驼峰、下划线法命名

### 驼峰：

1.  PHP 中的：变量、类、方法名

### 下划线：

1.  数据库

1.  接口传参（即 与数据库保持一致）