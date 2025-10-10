+++
title = "gdisk分区问题"
date = "2025-08-03T16:27:14+08:00"
summary = "gdisk分区最后保留2015个扇区没有使用的问题"
categories = ["硬盘分区"]
tags = ["gdisk", "gpt", "mbr"]
draft = false
+++

## 系统硬盘信息

使用qemu虚拟机安装debian系统，分配的硬盘大小为128G，系统的硬盘和分区信息如下：
```bash
Disk /dev/sda: 268435456 sectors, 128.0 GiB
Model: QEMU HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): A69CDA09-9C39-4696-8440-F7740B48D9B7
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 268435422
Partitions will be aligned on 2048-sector boundaries
Total free space is 4029 sectors (2.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         2099199   1024.0 MiB  EF00  EFI system partition
   2         2099200       268433407   127.0 GiB   8300  Linux filesystem
```

## 信息解读

- 128G硬盘有268435456个扇区，每个扇区大小512byte，计算后共128G，不多不少
- 扇区编号从0开始，即扇区范围：LBA 0-268435455
- GPT分区表会保留硬盘开头的34个扇区（LBA 0-33）
- 上面显示最后一个扇区为：268435422，而不是268435455，说明末尾保留了33个扇区，网上资料说是34个，网上资料应该是错误的
- 通过上面分析可得：GPT分区表会保留67个扇区
- GPT分区表第一个分区默认从第2049个扇区开始，即LBA 2048，硬盘最前面保留了1MB的空间
- 上面分区表最后一个扇区是LBA 268433407，268435455 - 268433407 = 2048，结尾也保留了1M的空间
- 硬盘剩余可用的4029个扇区，就是：4096 - 67 = 4029，这样来的
- gdisk默认2048个扇区对齐，即硬盘大小是MB的整数倍，因为硬盘末尾要保留33个分区，导致相邻的2015个扇区不足1MB，所以gdisk会保留末尾的2015个扇区
