---
title: Sublime 开启 Package Control 并加速
date: 2021-01-07 18:26:13
tags: Sublime
categories: 折腾
---

# 安装
## 方案 1：Command Palette
### Open the command palette
Win/Linux: ctrl+shift+p, Mac: cmd+shift+p
Type Install Package Control, press enter
### Menu
Open the Tools menu
Select Install Package Control…
This will download the latest version of Package Control and verify it using public key cryptography. If an error occurs, use the manual method instead.


## 方案 2：Manual
若方案 1 失败，可考虑本方案，通过手动的

If the command palette/menu method is not possible due to a proxy on your network or using an old version of Sublime Text, the following steps will also install Package Control:

1. Click the `Preferences` > `Browse Packages…` menu
2. Browse up a folder and then into the Installed Packages/ folder
3. Download [Package Control.sublime-package](https://packagecontrol.io/Package%20Control.sublime-package) and copy it into the Installed Packages/ directory
4. Restart Sublime Text

# 加速
1. Ctrl (Command)+Shift+p 打开命令面板。
2. 输入 Package Control: Add Channel，enter
3. 添加地址

在线：https://raw.githubusercontent.com/wilon/sublime/master/download/channel_v3.json
本地：/data/channel_v3.json

# 参考资料
https://packagecontrol.io/installation
https://www.cnblogs.com/mq0036/p/12945984.html
https://github.com/wilon/wilon.github.io/issues/8
https://blog.csdn.net/qq_39633494/article/details/93330323