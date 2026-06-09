+++
title = '使用飞牛虚拟机部署openwrt'
date = "2025-06-02T19:48:11+08:00"
summary = "使用qemu虚拟机部署openwrt"
categories = ["网络"]
tags = ["openwrt", "qemu", "飞牛", "虚拟机"]
draft = false
+++


## 镜像选择

新版本飞牛虚拟机已经支持，直接导入img、img.gz和qcow2格式的镜像，不用像以前，配置完，去磁盘位置做镜像替换。教程使用immortalwrt举例，使用下方的firmware-selector，先选择固件版本，然后去搜索x86，共有四种x86设备可以选择，一般选择Generic x86/64即可。
```
https://firmware-selector.immortalwrt.org
```
- Generic x86
	- 适用于32位x86系统
- Generic x86/64
	- 适用于64位x86系统
- Generic x86/Geode
	- 老旧硬件
- Generic x86/legacy
	- 老旧硬件

immortalwrt提供下载的qcow2镜像文件有下面四个，推荐使用squashfs固件，如果设置有问题，恢复出厂设置很方便，启动方式看个人喜好选择，文章以efi+squashfs固件举例。
- COMBINED-EFI (EXT4-COMBINED-EFI.QCOW2)
	- 通过efi方式启动，文件系统格式：ext4
- COMBINED (EXT4-COMBINED.QCOW2)
	- 通过传统bios方式启动，文件系统格式：ext4
- COMBINED-EFI (SQUASHFS-COMBINED-EFI.QCOW2)
	- 通过efi方式启动，文件系统格式：squashfs，很方便恢复出厂设置
- COMBINED (SQUASHFS-COMBINED.QCOW2)
	- 通过传统bios方式启动，文件系统格式：squashfs，很方便恢复出厂设置

## 新建虚拟机
- 填写虚拟机名称。操作系统选择Linux，内核版本选择6.x - 2.6。
- 导入下载的镜像。主板类型选择Q35(比i440fx更新的硬件)，启动方式选择UEFI，配置按照个人喜好就行。
- **添加网卡(重要)**：先添加的网口会作为openwrt系统中的lan口，最后一个网口会成为系统的wan口。可以先添加没有用到网口，最后添加飞牛正在使用上网的网口。
- 硬件直通，按需添加，不需要跳过即可。

## 开机
完成上面的操作后，就可以开机了，跑码停止的时候，按回车键，进入openwrt系统，输入ip addr，可以网口ip相关信息。
