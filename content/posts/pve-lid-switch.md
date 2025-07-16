+++
title = "pve系统下笔记本合盖锁屏不休眠"
date = "2025-07-17T00:53:55+08:00"
summary = "pve系统下笔记本合盖锁屏不休眠"
categories = ["虚拟机"]
tags = ["pve", "qemu"]
draft = false
+++

## 问题

老旧笔记本安装了pve系统，方便部署一些系统用于测试学习。遇到一个问题：合上笔记本盖子后，会导致系统休眠，服务不可用。

## 解决方案

修改系统文件，设置笔记本合上盖子，只锁屏(关闭屏幕灯)，系统不休眠。

编辑文件：

```bash
vim /etc/systemd/logind.conf 
```

修改下面的配置项：

```conf
# 笔记本合盖
HandleLidSwitch=lock
# 笔记本插电合盖
HandleLidSwitchExternalPower=lock
# 笔记本外接显示器合盖
HandleLidSwitchDocked=lock
```

上面选项的可选值：

- ignore（什么都不做）
- poweroff（关机）
- reboot（重新启动）
- halt（关机）
- suspend（待机挂起）
- hibernate（休眠）
- lock（锁屏，关闭屏幕灯）

重启服务：

```bash
systemctl restart systemd-logind
```
