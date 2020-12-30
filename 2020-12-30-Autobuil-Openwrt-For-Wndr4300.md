---
title: 为 WNDR4300 设置自动编译插件
date: 2020-12-30 22:51:30
tags: Openwrt
categories: 折腾
---

# 前言
刷了很多次 WNDR4300 固件了，每次更换都是因为原作者断更，以至于后续经常遇到路由器不稳定的情况时，也无法拿到最新版本的固件。所以决定自己编译。
本来想本地构建，然后发现构建环境需要搭建，习惯性的想到能不能用 docker，进而习惯性的想到能不能用线上的自动构建服务。比如阿里云的流水线，比如 Github 的 Actions。
# 前人种树
https://p3terx.com/archives/build-openwrt-with-github-actions.html
https://www.izheteng.site/2019/12/24/2019-12-24-%E4%BD%BF%E7%94%A8GitHub-Actions%E7%BC%96%E8%AF%91OpenWRT/
http://www.5lazy.cn/post-144.html
http://www.5lazy.cn/post-147.html
# 注意事项
- 期望切换 ar71xx 到 ath79。
- 开启 120M 内存空间。
- 切换主题到最新版。

```
sed -i s/'23552k(ubi),25600k@0x6c0000(firmware)'/'120832k(ubi),122880k@0x6c0000(firmware)'/ target/linux/ar71xx/image/legacy.mk
sed -i s/'luci-theme-bootstrap'/'luci-theme-argon'/ feeds/luci/collections/luci/Makefile
sed -i s/"timezone='UTC'"/"timezone='Asia\/Shanghai'"/  package/base-files/files/bin/config_generate
```
# 详细参数
```
CONFIG_TARGET_ath79=y
CONFIG_TARGET_ath79_nand=y
CONFIG_TARGET_ath79_nand_DEVICE_netgear_wndr4300=y
CONFIG_LIBCURL_COOKIES=y
CONFIG_LIBCURL_FILE=y
CONFIG_LIBCURL_FTP=y
CONFIG_LIBCURL_HTTP=y
CONFIG_LIBCURL_MBEDTLS=y
CONFIG_LIBCURL_NO_SMB="!"
CONFIG_LIBCURL_PROXY=y
CONFIG_PACKAGE_blkid=y
CONFIG_PACKAGE_boost=y
CONFIG_PACKAGE_boost-date_time=y
CONFIG_PACKAGE_boost-program_options=y
CONFIG_PACKAGE_boost-system=y
CONFIG_PACKAGE_ca-bundle=y
CONFIG_PACKAGE_curl=y
CONFIG_PACKAGE_dmesg=y
CONFIG_PACKAGE_fdisk=y
CONFIG_PACKAGE_file=y
CONFIG_PACKAGE_ipt2socks=y
CONFIG_PACKAGE_iptables-mod-conntrack-extra=m
CONFIG_PACKAGE_iptables-mod-ipopt=m
CONFIG_PACKAGE_kcptun-client=y
CONFIG_PACKAGE_kmod-ipt-conntrack-extra=m
CONFIG_PACKAGE_kmod-ipt-ipopt=m
CONFIG_PACKAGE_kmod-nls-utf8=y
CONFIG_PACKAGE_kmod-scsi-core=y
CONFIG_PACKAGE_kmod-usb-storage=y
CONFIG_PACKAGE_kmod-usb-storage-extras=y
CONFIG_PACKAGE_kmod-usb-uhci=y
CONFIG_PACKAGE_libblkid=y
CONFIG_PACKAGE_libbz2=y
CONFIG_PACKAGE_libcurl=y
CONFIG_PACKAGE_libfdisk=y
CONFIG_PACKAGE_liblzma=y
CONFIG_PACKAGE_libmagic=y
CONFIG_PACKAGE_libmount=y
CONFIG_PACKAGE_libncurses=y
CONFIG_PACKAGE_libsmartcols=y
CONFIG_PACKAGE_libstdcpp=y
CONFIG_PACKAGE_lsblk=y
CONFIG_PACKAGE_luci-app-mwan3=m
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Kcptun=y
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Trojan=y
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_V2ray_plugin=y
CONFIG_PACKAGE_luci-i18n-mwan3-zh-cn=m
CONFIG_PACKAGE_mount-utils=y
CONFIG_PACKAGE_mwan3=m
CONFIG_PACKAGE_terminfo=y
CONFIG_PACKAGE_trojan=y
CONFIG_PACKAGE_v2ray-plugin=y
CONFIG_PACKAGE_vim=y
CONFIG_PACKAGE_zoneinfo-asia=y
CONFIG_boost-compile-visibility-hidden=y
CONFIG_boost-runtime-shared=y
CONFIG_boost-static-and-shared-libs=y
CONFIG_boost-variant-release=y
```