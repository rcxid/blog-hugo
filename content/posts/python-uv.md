+++
title = "使用uv安装管理python"
date = "2025-07-10T15:15:05+08:00"
summary = "使用uv安装管理python"
categories = ["开发工具"]
tags = ["python"]
draft = false
+++

## 前言

macos自带的python版本有点低，我的电脑上默认安装的python版本是3.9。安装某些python依赖时，已经无法满足最低python版本要求了。我通过折腾python开发环境，发现在macos中使用uv安装管理python更方便。

## 通过uv安装python

1、首先使用pip3安装一下uv，或者通过官网文档安装一下：https://docs.astral.sh/uv/getting-started/installation/
```bash
pip3 install --upgrade uv
```

2、查看可以安装的python版本
```
uv python list
```

3、安装python3.11
```bash
uv python install 3.11
```

4、使用uv创建python项目，指定python版本3.11
```bash
uv init --python 3.11 <project-name>
```

5、创建python虚拟环境，指定python版本3.11
```bash
uv venv --python 3.11
```
