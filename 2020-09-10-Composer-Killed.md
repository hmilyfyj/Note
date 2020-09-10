---
title: Composer 执行时被 Killed
date: 2020-09-10 19:24:20
tags: PHP
categories: PHP
---

最近在执行执行 `composer require` 时出现了执行失败的情况，控制台返回 `Killed`，没有其他报错。是被 `oom-killer` 干掉了。解决方法是提高内存。

- 如果是命令是在普通环境执行，可以通过增加 swap 的方法。
- 如果是在 docker 容器中被 kill，可以通过提高分配给 docker 的内存。

![](/media/15997374759676.jpg)
