---
title: Redmi Ax5 开启 SSH 和富强
date: 2020-12-31 21:06:45
tags: Openwrt,AX5
categories: 折腾
---
# 现存问题
偶尔会断网，不清楚啥原因。
# 开启 ssh

1. 获取 stok。
![](/media/16094205274693.jpg)
1. 替换 stok 到下方的 <STOK> 并访问该网址，即可重置 root 密码为 admin。
```
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20nvram%20set%20ssh_en%3D1%3B%20nvram%20commit%3B%20sed%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%5C%22debug%5C%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%3B%20%2Fetc%2Finit.d%2Fdropbear%20start%3B%20echo%20-e%20'admin%5Cnadmin'%20%7C%20passwd%20root%3B
```

# 安装 clash

Github：https://github.com/juewuy/ShellClash

```
opkg update && opkg install curl

# 安装 clash
#Release version - by github
sh -c "$(curl -kfsSl --resolve raw.githubusercontent.com:443:199.232.68.133 https://raw.githubusercontent.com/juewuy/ShellClash/master/install.sh)" && source /etc/profile &> /dev/null
#Release version - by jsdelivrCDN
sh -c "$(curl -kfsSl https://cdn.jsdelivr.net/gh/juewuy/ShellClash@master/install.sh)" && source /etc/profile &> /dev/null
#Test version - by github
sh -c "$(curl -kfsSl --resolve raw.githubusercontent.com:443:199.232.68.133 https://raw.githubusercontent.com/juewuy/ShellClash/master/install.sh)" -s 1 && source /etc/profile &> /dev/null
```

# 配置 clash
启动 clash 时，需要输入订阅链接。订阅链接的作用是利用转换服务把酸酸乳转为 clash。构造好地址后访问，把配置信息放入 `/etc/clash/config.yaml` 中即可，不要在 console 中填写地址，太慢了，还总是更新失败。

假如你测试多次后发现 clash 无法启动了，报错字样中油 `/etc/clash/mark`，说明 bug 了，把路由器中该文件删除即可。

```
http://127.0.0.1:25500/sub?target=%TARGET%&url=%URL%&config=%CONFIG%

# 参数说明
TARGET：clash。
URL：订阅地址、节点地址即可
CONFIG：https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_MultiMode.ini

# 成品地址
http://localhost:25500/sub?target=clash&new_name=true&url=URL&insert=false&config=https%3A%2F%2Fraw.githubusercontent.com%2FACL4SSR%2FACL4SSR%2Fmaster%2FClash%2Fconfig%2FACL4SSR_Online_Full_MultiMode.ini
```

http://localhost:25500 是本地构建的转换服务：`docker run -d --restart=always -p 25500:25500 tindy2013/subconverter:latest`

参考资料：
https://github.com/CareyWang/sub-web
https://github.com/tindy2013/subconverter/blob/master/README-cn.md