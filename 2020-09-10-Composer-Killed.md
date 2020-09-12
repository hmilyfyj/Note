---
title: Composer 执行时被 Killed
date: 2020-09-10 19:24:20
tags: PHP,PHPUnit
categories: PHP
---

最近在执行执行 `composer require` 时出现了执行失败的情况，控制台返回 `Killed`，没有其他报错。是被 `oom-killer` 干掉了。解决方法是提高CPU 或~~内存~~。

- ~~如果是命令是在普通环境执行，可以通过增加 swap 的方法。~~ CPU 不足，所以提高内存没用。
- 如果是在 docker 容器中被 kill，可以通过提高分配给 docker 的~~内存~~、CPU。Mac 下的配置方法如图。

![](/media/15997374759676.jpg)
