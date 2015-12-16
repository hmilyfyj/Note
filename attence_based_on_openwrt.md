title: 基于Openwrt的手机考勤
date: 2015-12-6 22:36
tags: []
categories: 
---

设备：wndr4300

### 计划任务
功能：获取设备列表，并上传到服务器

    cat /proc/net/arp | grep 0x2 > /device.txt
    curl -H "Expect:" -F "userfile=@/device.txt"  http://at.xuyu.club/attence

