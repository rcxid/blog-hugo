+++
title = '使用飞牛虚拟机部署openwrt'
date = "2025-06-02T19:48:11+08:00"
summary = "使用qemu虚拟机部署openwrt"
categories = ["网络"]
tags = ["openwrt", "qemu", "飞牛", "虚拟机"]
draft = false

[cover]
image = "文章封面图"
+++


## 下载openwrt镜像
immortalwrt提供qcow2镜像下载，教程使用immortalwrt举例，openwrt官方镜像可以使用qemu-img工具把img镜像转成qcow2再使用。使用下方的firmware-selector，先选择固件版本，然后去搜索x86，共有四种x86设备可以选择，一般选择Generic x86/64即可。
```
https://firmware-selector.immortalwrt.org
```
- Generic x86
	- 适用于32位x86系统
- Generic x86/64
	- 适用于64位x86系统
- Generic x86/Geode
	- 老旧硬件，不太了解
- Generic x86/legacy
	- 老旧硬件，不太了解

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
填写虚拟机名称。操作系统选择Linux，内核版本选择6.x - 2.6
![1749138183654.png](https://img.617495.xyz/2025/06/05/6841bb08c1135.png)
可以在本机创建一个空的iso文件，选择这个空的iso文件就行，后面用不到。主板类型选择Q35(比i440fx更新的硬件)，启动方式选择UEFI，配置按照个人喜好就行。
![1749138312990.png](https://img.617495.xyz/2025/06/05/6841bb8e5b462.png)
添加存储空间，大小随便写，后面用不到。
![1749138523547.png](https://img.617495.xyz/2025/06/05/6841bc5cf188e.png)
**添加网卡(重要)**：先添加没有用到网口，最后添加飞牛使用上网的网口，这个网口顺序，关系到openwrt中的lan和wan口，最后一个网口会成为系统的wan口，因此最后添加飞牛使用上网的网口。
![1749138611274.png](https://img.617495.xyz/2025/06/05/6841bcb45e79a.png)
硬件直通，按需添加，不需要跳过即可。

## 编辑虚拟机
移除上面挂着的iso镜像
![6841be01645ea.png](https://img.617495.xyz/2025/06/05/6841be01645ea.png)
查看虚拟机详情，如图所示，查看磁盘镜像位置，将上面下载的immortalwrt镜像替换上面的文件即可。
![6841c41b567ed.png](https://img.617495.xyz/2025/06/06/6841c41b567ed.png)

## 开机
完成上面的操作后，就可以开机了，跑码停止的时候，按回车键，进入openwrt系统，输入ip addr，可以看到eth1成功获取到了路由器ip，eth1就是wan口，eth0就是lan口。

![1749140107299.png](https://img.617495.xyz/2025/06/06/6841c28c69716.png)

## 改进之处
推荐飞牛虚拟机官方，支持直接导入qcow2镜像创建系统，就像virt-manager那样，上面就不需要再编辑系统了。