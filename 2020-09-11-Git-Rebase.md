---
title: Git Rebasae
date: 2020-09-11 19:37:02
tags: Git
categories: Git
---

中文译名变基，顾名思义，把某个指定的提交，变为当前分支的上一层提交。

使用场景可以是：
- 移除无用的 commit 纪录。
- 合并两个无关联的分支。

相关命令：
- 退出变基过程：`git rebase --abort`。
- 继续变基：git rebase --continue
- 清空当前未提交的内容：https://www.jianshu.com/p/10f4d811985e
    - git clean -xdf
    - git reset --hard

- 创建空分支：git checkout --orphan emptybranch

使用过程中遇到的问题：