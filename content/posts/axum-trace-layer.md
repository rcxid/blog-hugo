---
title: axum通过layer为日志添加用户信息
subtitle: 
date: 2025-06-23T21:56:40+08:00
slug: fad81e1
draft: false
author:
  name: 
  link: 
  email: 
  avatar: 
description: 
keywords: axum 中间件 日志
license: 
comment: true
weight: 0
tags:
  - axum
  - layer
categories:
  - 后端开发
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRelated: false
hiddenFromFeed: false
summary: 
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password: 
message: 
repost:
  enable: true
  url:
---

## 前言
最近遇到一个问题，之前使用rust axum开发的后端程序，排查线上问题的时候，发现日志中没有包含用户信息，因此无法通过用户id把需要的日志挑出来。因为之前在本地开发，只有我一个用户，所以没日志有没有用户id不会影响我排查问题。我想不修改日志打印代码的前提下，后续打印日志，日志自动加上用户信息。

## 解决方案探索
最开始，我通过deepseek询问问题的解决方法，deepseek给出的解决方案是通过自定义layer中间件 + 自定义FormatEvent，自定义layer中间件中记录的用户信息，在FormatEvent中把记录的信息打印出来。结果是AI给出的代码，我测试了2天，发现一堆问题，最后发现这个方案根本不行。

后面我开始在自定义layer中间件中使用span!宏记录用户信息，在 Rust 的`tracing`生态系统中，`span!`宏是构建分布式追踪系统的核心工具。它允许你创建一个**操作范围**（span），用于记录程序执行过程中的上下文信息和时间消耗。后续日志中的确包含了添加的用户信息，但是存在用户信息叠加问题，我感觉应该是span生命周期导致的。

## 最终解决方案
最后的解决方案，还是通过layer解决的。使用的是tower_http的TraceLayer中间件，在TraceLayer的make_span_with方法中，将request header中的token信息提取解密后，通过span!宏进行记录，后续打印的日志中包含了自己添加的用户信息。
### 参考代码
```rust
use crate::extractor::jwt::DecryptClaims;
use axum::body::Body;
use axum::http::Request;
use tower_http::trace::{HttpMakeClassifier, TraceLayer};
use tracing::Span;

/// 添加用户信息layer
pub fn add_user_info_layer() -> TraceLayer<HttpMakeClassifier, fn(&Request<Body>) -> Span> {
    TraceLayer::new_for_http().make_span_with(|request: &Request<Body>| {
        if let Some(Some(token)) = request
            .headers()
            .get("authorization")
            .map(|x| x.to_str().ok().map(|y| y["Bearer ".len()..].trim_start()))
        {
            if let Ok(claims) = DecryptClaims::try_from_token(token, false) {
                return tracing::info_span!("", openid = %claims.openid, user_id = %claims.id);
            }
        }
        Span::current()
    })
}
```
