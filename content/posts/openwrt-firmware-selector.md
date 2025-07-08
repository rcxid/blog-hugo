+++
title = "使用 Firmware Selector 构建openwrt镜像"
date = "2025-07-08T15:02:20+08:00"
summary = "使用 Firmware Selector 构建openwrt镜像"
categories = ["路由器"]
tags = ["openwrt"]
draft = false
+++

## 1、Firmware Selector地址

openwrt：
https://firmware-selector.openwrt.org
immortalwrt：
https://firmware-selector.immortalwrt.org

## 2、自定义预安装软件包

```bash
# 系统缺失中文语言包
luci-i18n-base-zh-cn
luci-i18n-package-manager-zh-cn
luci-i18n-firewall-zh-cn

# 系统升级
luci-i18n-attendedsysupgrade-zh-cn

# 文件网络共享相关
luci-i18n-samba4-zh-cn

# 硬盘休眠
luci-i18n-hd-idle-zh-cn

# 文件系统
kmod-fs-xfs

# 上网相关(最后两个immortalwrt才有)
luci-i18n-v2raya-zh-cn v2ray-geoip v2ray-geosite
luci-i18n-passwall-zh-cn
luci-app-openclash

# 其他工具
lsblk
luci-i18n-ttyd-zh-cn
```

## 3、自定义首次启动脚本
### 3.1、uci 命令修改默认lan ip

openwrt默认的 lan ip 为192.168.1.1，一般会和家中的上游光猫网络ip冲突。
```bash
uci set network.lan.ipaddr=192.168.3.1
uci commit network
```

## 4、uci 官方文档参考

https://openwrt.org/zh/docs/guide-user/base-system/uci
