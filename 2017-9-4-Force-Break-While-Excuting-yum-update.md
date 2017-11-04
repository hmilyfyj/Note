---
title: 意外中断 yum update 后的处理措施。
date:  2017-09-03 1:3:9
tags: Note
categories: Note
grammar_cjkRuby: true
---

更新（`yum update`）过程中不小心按了`Ctrl + C`

<!-- more -->

---

``` shell
yum install yum-utils
yum-complete-transaction --cleanup-only
#清除可能存在的重复包
package-cleanup --dupes
#清除可能存在的损坏包
package-cleanup --problems
#清除重复包的老版本：
package-cleanup --cleandupes
```


参考资料：
http://blog.chinaunix.net/uid-21710705-id-3039675.html
https://superuser.com/questions/243460/what-to-do-when-ctrl-c-cant-kill-a-process
http://blog.163.com/zhoutao_1001/blog/static/979024220122103352022/
http://blog.csdn.net/delph123456/article/details/17414115