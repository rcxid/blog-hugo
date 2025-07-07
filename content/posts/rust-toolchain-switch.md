+++
title = "rust stable和nightly版本切换"
date = "2025-07-07T16:40:39+08:00"
summary = "rust stable和nightly版本切换"
categories = ["开发工具"]
tags = ["rust"]
draft = false
+++

## 开始
rust语言一些特性需要是nightly版本才支持，rust默认安装的是stable版本，这里介绍一下如何快速切换到nightly版本。

## 使用rustup查看已经安装版本
```shell
rustup toolchain list
```
输出结果：stable-aarch64-apple-darwin (active, default)

## 安装并切换nightly版本

```shell
rustup default nightly
```

## 查看rustc版本，验证是否切换成功

```sh
rustc --version
```

输出结果举例：rustc 1.89.0-nightly (d13a431a6 2025-06-09)

看到nightly表示已经切换成功

## 切换到stable版本

```sh
rustup default stable
```

结束
