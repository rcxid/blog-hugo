+++
title = "安装windows时使用diskpart分区"
date = "2025-07-07T16:55:59+08:00"
summary = "安装windows时使用diskpart分区"
categories = ["系统安装"]
tags = ["windows"]
draft = false
+++

## 步骤
1. windows安装界面，按下Shift+F10调出命令提示符
2. 输入diskpart命令后按回车键，进入DiskPart工具
3. 输入list disk命令后按回车键，查看电脑当前可用硬盘
4. 输入select disk x（x为上述硬盘编号），选择你要进行分区操作的硬盘
5. 输入clean命令清除该硬盘上的所有分区（注意硬盘所有数据会丢失）
6. 输入convert gpt命令将该硬盘转换成GPT分区表
7. 创建EFI分区，执行create partition efi size=1024（分区大小为1024MB）
8. 格式化EFI分区，format quick fs=fat32 label=System
9. 创建MSR分区，执行create partition msr size=128（微软系统保留分区，分区大小为128MB）
10. 创建主分区（可选），执行create partition primary size=xxx（具体大小根据你的要求而定，作为系统分区来说，如果有足够空间，可以留出大于100GB即102400MB的空间，命令为create partition primary size=102400，方便系统有足够周转空间）
11. 关掉命令cmd，选择上面创建的主分区安装windows，或者界面上创建分区按照windows。