title: 基于Openwrt的考勤
date: 2015-12-22 22:31
tags: [openwrt,折腾]
categories: 折腾
---

设备：wndr4300

###  路由端

### Wndr4300

功能：获取当前设备列表，并上传到服务器

    cat /proc/net/arp | grep 0x2 > /device.txt
    curl -H "Expect:" -F "userfile=@/device.txt"  http://at.xuyu.club/attence

### 服务器端

1. Mac地址与人员的绑定

### 可能出现的问题

1. 同一MAC出现两次。
