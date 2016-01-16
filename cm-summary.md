title:  CM玩耍技巧、踩坑集锦
date: 2016-01-16 08:19
tags: [cm]
categories: mobile
---

错误提示：`Read-only file system.`

解决：

Simply change ro to rw and add the remount option

    # mount -o rw,remount /system

Once you are done making changes, you should remount with the original readonly.

    # mount -o ro,remount /system

[原文链接](http://stackoverflow.com/questions/6066030/read-only-file-system-on-android)


### 更新 Hosts

wget -c http://smarthosts.googlecode.com/svn/trunk/hosts >> /etc/hosts
