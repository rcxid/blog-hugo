+++
title = "Tmux 相关文档"
date = "2025-07-08T15:45:36+08:00"
summary = "记录 tmux 学习"
categories = ["工具"]
tags = ["tmux", "terminal"]
draft = false
+++

## 开始

**Tmux** 是一个终端复用器（terminal multiplexer），它允许用户在一个终端窗口中访问多个独立的会话。**Tmux** 默认快捷键(prefix) 是 ctrl + b，我一般会修改成ctrl + f。

## 简单配置示例

```conf
unbind C-b
set -g prefix C-f
set -g mouse on

set -g default-terminal "tmux-256color"
set-option -a terminal-overrides ",*256col*:RGB"

set -g base-index 1
set -g pane-base-index 1
set -g renumber-windows on
```

## 常用快捷键

```bash
# 创建一个新session
tmux new -s <session-name>

# 创建窗口
prefix + c

# 窗口列表展示
prefix + w

# detach
prefix + d

# 查看tmux session list
tmux ls

# detach后重新进入tmux session
tmux attach -t <session-name>

# 切换窗口，上一个，下一个
prefix + p/n

# 竖分屏，左右分屏
prefix + %

# 横分屏，上下分屏
prefix + "
```
