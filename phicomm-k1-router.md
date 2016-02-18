

title: 路由器-折腾日志
date: 2016-02-18 15:26
tags: [折腾,路由]
categories: 路由


---

家里新买了路由器，折腾一下。

<!-- more -->

# 一、刷Breed

## 免tftp原厂刷breed

### 免tftp原厂刷breed方式：
1. 先接好路由,让路由能上网;
2. 查看自己的电脑ip地址,设定电脑的ip是192.168.2.100;（懂得看自己电脑IP的可略过）
3. 在浏览器上登陆路由器,192.168.2.1,账号:admin,密码:admin;
4. 浏览器地址栏打开
  http://192.168.2.1/goform/Diagnosis?pingAddr=192.168.2.100|echo""|telnetd
(红色部分是本机ip,如果你的电脑ip不是192.168.2.100,请改成你电脑的ip);
5. 点击 开始菜单->运行,输入cmd,打开控制台;
6. 

    telnet 192.168.2.1
    账号:admin,密码:admin;

7. `wget http://breed.hackpascal.net/breed-mt7620-reset1.bin`

8.  `mtd_write write breed-mt7620-reset1.bin Bootloader`

9. 写入完成后,拔掉路由电源，按住reset按钮---插上电---看到所有灯闪后，松开reset。（电脑IP改回192.168.1.100）浏览器进去192.168.1.1访问breed界面。

10. 到‘更新固件’里面选择固件刷进路由即可。
 


![enter image description here](http://www.right.com.cn/forum/forum.php?mod=attachment&aid=MTIwNDQ2fDJiNzVlN2MwfDE0NTU3NzkxOTl8MHwxNzg3MDc=&noupdate=yes)


# 二、多拨

默认帐号：`root:admin`

1. 打开网络-接口-WAN-一般设置将WAN口协议切换成PPPoE，然后点击切换协议
2. 切换到“高级设置”选项卡，设置网关跃点为40。点击保存&应用
3. 网络-虚拟WAN接口，勾选启动；虚拟WAN接口数量写5；勾选断线重连；最低在线接口数写1。点击保存&应用


