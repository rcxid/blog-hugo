+++
title = "axum默认body大小导致图片上传失败问题"
date = "2025-08-10T16:35:37+08:00"
summary = "开发图床后端的图片上传接口遇到的坑"
categories = ["后端开发"]
tags = ["axum", "middleware", "layer"]
draft = false
+++

## 问题

我最近在开发一个图床程序，后端使用rust axum构建，刚开发完时，测试正常上传图片。今天初步写了下前端上传界面，使用前端界面上传的时候，上传图片总失败，后端接口处报错。

### 排除过程

- 因为前端我使用了ant design的上传组件，我感觉是组件对form数据处理有问题，于是调整为手动上传，还是报错。
- 因为调用后端接口的代码是我使用的openapi-ts库生成的，于是我又改成通过fetch方法调用接口，同样报错。
- 我开始怀疑后端是不是依赖冲突导致的，因为axum使用的bytes库和我引用的bytes库版本相差很大，后面测试发现不是这个原因，rust依赖冲突，很多情况下编译期都通不过。
- 我怀疑是不是图片格式有问题，于是把图片从png转成jpg，结果还是报错。
- 我自己又截图生成一张图片，上传是成功的。这个时候，我开始怀疑是图片大小导致的。

## 解决问题

我搜索了一下axum如何配置http body大小的相关资料，找到一篇文章，文章中提到了一个结构体：ContentLengthLimit，这个结构体可以设置http body的大小。我自己引入到时候，发现没有这个结构体，去github代码仓库搜索这个结构体时，官方提示：ContentLengthLimit已经废弃了，提示使用：DefaultBodyLimit layer去设置，然后我去查看DefaultBodyLimit::max(...)方法时，发现注释中说axum的body默认最大为2MB，因此上传大文件时，就会失败。假设得到验证。

路由配置参考代码
```rust
Router::new()
    .route("/", get(root))
    .nest("/api", api_router)
    // 设置body大小最大为16MB
    .layer(DefaultBodyLimit::max(16 * 1024 * 1024 * 1024));
```
