+++
title = "IMX6ULL学习笔记（持续更新）"
date = "2025-07-25T16:01:31+08:00"
summary = "linux开发学习"
categories = ["嵌入式"]
tags = ["imx6ull", "linux"]
draft = false
+++

## 一、汇编相关

为什么学Cortex-A汇编
- 使用汇编初始化一些 SOC 外设
- 初始化 DDR， IMX6U不需要
- 设置 sp 指针，一般指向 DDR，设置好C语言运行环境

开发版 LED 灯硬件原理分析

思路：控制 LED 灯连接的端口输出电平高低，控制 LED 的亮和关

I.MX6ULL IO初始化
- 使能时钟
- IO 复用
- 设置电气属性

LDR指令：从存储器加载数据到寄存器

STR指令：将寄存器中的数据写到存储器


```bash
int a, b;
a = b;

// a: 0x20 b: 0x30


LDR R0, =0x30
SRT R1, [R0]

LDR R0, =0x20
SRT R1, [R0]

# 编译汇编代码
arm-linux-gnueabihf-gcc -g -c led.s -o led.o

6ULL
内部RAM：0x900000～0x91FFFF
512DDR：0x80000000～0x9FFFFFFF
256DDR：0x80000000～0x8FFFFFFF
        0x87800000

# 链接成elf文件
arm-linux-gnueabihf-ld -Ttext 0x87800000 led.o -o led.elf

# 生成bin文件
arm-linux-gnueabihf-objcopy -O binary -S -g led.elf led.bin

# 反汇编
arm-linux-gnueabihf-objdump -D led.elf > led.dis
```
