+++
title = "UDF类污染问题"
date = "2025-07-17T21:23:05+08:00"
summary = "UDF函数通过包名重定位，解决类污染问题"
categories = ["数据开发"]
tags = ["udf", "hive", "spark", "flink"]
draft = false
+++


## 前言

java项目打包时，将项目代码路径（包名）从 `com.xxx.yyy` 修改为 `prefix.com.xxx.yyy` 的操作，称为**包名重定位（Package Relocation）** 或**类路径重映射（Classpath Remapping）**。

## UDF类污染问题

这个问题是我工作中发现的，之前记录在公司内部文档，现在整理一下，写到自己博客记录一下。问题是，我在一个flink任务中，更新了udf函数，但是udf函数的输出不符合预期，通过排查，我发现，这个任务用到了两个udf，两个udf函数是从同一个项目打包的，因为涉及的工具类新旧版本不一致，导致的类污染，旧版本的类被加载，因此函数更新后，逻辑没有生效。

![1752918043320.png](https://img.617495.xyz/2025/07/19/687b681d07bf5.png)

## 解决方案

通过**包名重定位**可以解决上面的问题，例如UDF_A打包的时候，使用**包名重定位**，将项目代码中代码包名从`com.xxx.yyy` 修改为 `vyyyyMMddHHmm.com.xxx.yyy`，原理如下图所示。

![1752918388525.png](https://img.617495.xyz/2025/07/19/687b6975d2e38.png)

## gradle实现

gradle可以使用shadow插件实现包名重定位。shadow对gradle版本有要求，以gradle7.x和shadow7.x举例，配置代码如下。

```kotlin
// rootProject配置
buildscript {
    repositories {
        gradlePluginPortal()
    }
    dependencies {
        classpath("com.github.johnrengelman.shadow:com.github.johnrengelman.shadow.gradle.plugin:7.1.2")
    }
}

subprojects {
    apply(plugin = "java")
    apply(plugin = "com.github.johnrengelman.shadow")
}

// 自定义函数模块配置
import java.time.LocalDateTime
import java.time.format.DateTimeFormatter

val buildTime: String = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmm"))

tasks.shadowJar {
    relocate("org.example.udf", "v${buildTime}.org.example.udf")
    relocate("org.example.util", "v${buildTime}.org.example.util")
}
```

## 代码效果

使用shadowJar打包后，代码包名会自动加上时间前缀，以后代码再也不会冲突了。

![1752921193084.png](https://img.617495.xyz/2025/07/19/687b746a2392c.png)