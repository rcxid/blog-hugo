+++
title = "macos使用UTM虚拟机安装armbian"
date = "2025-04-28T15:36:51+08:00"
summary = "macos使用UTM虚拟机安装armbian"
categories = ["工具"]
tags = ["虚拟机", "qemu", "armbian"]
draft = false
+++

## 安装UTM虚拟机

官网地址：[https://mac.getutm.app](https://mac.getutm.app)

## 下载Armbian镜像

armbian官网：[https://www.armbian.com](https://www.armbian.com)

### 官方qcow2镜像启动失败

UTM虚拟机支持qcow2镜像快速启动，相比iso安装使用更方便。下面日期编译的armbian镜像测试后，发现无法正常启动。
![1752158797543.png](https://img.617495.xyz/2025/07/10/686fd2509b9c3.png)
### 使用armbian官方img镜像转qcow2格式
查看img镜像信息
```bash
qemu-img info Armbian_25.2.3_Uefi-arm64_bookworm_current_6.12.20_minimal.img
```
![1752158968952.png](https://img.617495.xyz/2025/07/10/686fd2faba1df.png)

将img镜像转qcow2格式
```bash
qemu-img convert -f raw -O qcow2 -c Armbian_25.2.3_Uefi-arm64_bookworm_current_6.12.20_minimal.img Armbian_25.2.3_Uefi-arm64_bookworm_current_6.12.20_minimal.qcow2
```
- `-f raw`指定输入文件格式为raw（img通常为raw格式）
- `-O qcow2`指定输出格式为qcow2
- `-c`表示压缩输出qcow2文件大小
- `Armbian_25.2.3_Uefi-arm64_bookworm_current_6.12.20_minimal.img`是输入的img文件
- `Armbian_25.2.3_Uefi-arm64_bookworm_current_6.12.20_minimal.qcow2`是输出的qcow2文件

## UTM安装armbian
### 1、选择虚拟化
![1752159544759.png](https://img.617495.xyz/2025/07/10/686fd539c3617.png)
### 2、操作系统选择其他
![1752159607400.png](https://img.617495.xyz/2025/07/10/686fd5789ef37.png)
### 3、启动设备选择无，取消勾选旧版硬件
![1752159634470.png](https://img.617495.xyz/2025/07/10/686fd5940cd7a.png)
### 4、调整硬件资源
![1752159672606.png](https://img.617495.xyz/2025/07/10/686fd5ba8bcdc.png)
### 5、设置硬盘大小
硬盘大小随意，这里的设置不会生效，可直接下一步
![1752159711641.png](https://img.617495.xyz/2025/07/10/686fd5e0aab1e.png)
### 6、设置共享目录
![1752159757861.png](https://img.617495.xyz/2025/07/10/686fd60ec7b13.png)
### 7、勾选打开虚拟机设置
配置一下虚拟机名称，勾选打开虚拟机设置，继续配置虚拟机。
![1752159800996.png](https://img.617495.xyz/2025/07/10/686fd63a26c4e.png)
### 8、添加串行设备
如果安装的是不带桌面环境的armbian，可以添加串行去显示终端，显示更清晰。删除多余的显示和声音设备。
![1752159879972.png](https://img.617495.xyz/2025/07/10/686fd68986f6d.png)
### 9、网络配置
可以选择桥接（虚拟机ip和mac在同一网段下），方便后续连接通信，桥接网口，选择自己常用的（macmini m2，网口是en0，wifi是en1，具体可以看系统信息app中查找相关信息）
![1752159930496.png](https://img.617495.xyz/2025/07/10/686fd6bc1e44e.png)
### 10、导入系统镜像
删除默认创建的VirtIO驱动器，新建导入qcow2格式镜像，在调整大小处，调整需要的磁盘大小。
![1752159964917.png](https://img.617495.xyz/2025/07/10/686fd6df1271c.png)
### 11、启动虚拟机，完成安装
保存配置，启动虚拟机，创建root密码，创建普通用户，配置地区。完成安装。
![1752160005417.png](https://img.617495.xyz/2025/07/10/686fd70679af4.png)
