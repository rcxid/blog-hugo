+++
title = "qemu虚拟机使用总结"
date = "2025-06-06T21:53:20+08:00"
summary = "qemu常用命令相关"
categories = ["虚拟机"]
tags = ["qemu", "qcow2", "虚拟机"]
draft = false
+++

## 介绍

qcow2的全称是：QEMU Copy-On-Write version 2，是qemu开发的一种高性能、功能丰富的磁盘镜像格式。qemu虚拟机安装十分方便：主流linux可直接通过包管理器安装，且qemu的kvm在linux系统支持良好，使用kvm后，虚拟机的性能损耗极低。

## qcow2优点

- 轻量化存储：磁盘镜像文件刚开始很小，会动态化增长
- 快照管理：快速创建快照，方便恢复
- 高性能
- 系统备份迁移方便

## linux下安装qemu虚拟机

```bash
# 以debian和ubuntu举例
sudo apt install qemu-system-x86 qemu-utils ipxe-qemu virt-manager
```

## qemu常用命令

```bash
# 查看镜像信息
qemu-img info xxx.img
qemu-img info xxx.qcow2
# 对qcow2镜像进行扩容
qemu-img resize xxx.qcow2 128G
# raw img转qcow2格式
qemu-img convert -f raw -O qcow2 -c input.img output.qcow2
```

## 使用cloud image快速部署系统

使用cloud image + cloud-init可以实现系统的快速部署，省去一些繁琐的人工安装步骤。debian和ubuntu官方都提供cloud image下载，在系统不是pve虚拟机的情况下，使用cloud-init有一定门槛，可以选择armbian，armbian在首次开机时，通过用户交互输入完成系统的一些初始化配置，也可以快速部署系统。可以按照自己cpu架构选择下载对应架构的armbian镜像文件。
```bash
# 注意安装前调整armbian镜像文件系统大小，系统首次初始化，会自动扩容，使用全部的空间
qemu-img resize armbian.qcow2 128G
# x86-64
https://www.armbian.com/uefi-x86
# arm64
https://www.armbian.com/uefi-arm64
```

armbian存在的问题，部署k8s集群，请不要使用armbian镜像，有些驱动，armbian无法加载，这些驱动，貌似是k8s必须的。armbian虽然基于debian或者ubuntu，但是通过debian/ubuntu iso部署的系统和armbian镜像部署的系统还是存在差异。
