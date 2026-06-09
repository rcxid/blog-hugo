+++
date = "2025-06-23T21:56:40+08:00"
title = "axum通过layer为日志添加自定义信息"
summary = "axum通过layer为tracing日志添加自定义信息"
categories = ["后端开发"]
tags = ["axum", "middleware", "tower-http", "TraceLayer"]
draft = false
+++

## 前言
使用axum做后端开发过程中，部分接口会使用到layer中间件。layer中间件可以给接口添加前置和后置的拦截逻辑，并且不需要修改接口逻辑代码。最近遇到一个需求，发现日志打印也可以通过tower-http的TraceLayer实现一些自定义信息的添加，并且不需要修改原来打印日志代码。文章以添加用户信息举例。

## 解决方案探索
最开始，我通过deepseek询问问题的解决方法，deepseek给出的解决方案是通过自定义layer中间件 + tracing的自定义FormatEvent，自定义layer中间件中记录的用户信息，在FormatEvent中把记录的信息打印出来。这个方案应该是可行的，但是代码量太大，后面测试发现，还有一些问题，就没有采用这种方案。这个方案中使用到了span!宏记录用户信息。在 Rust 的tracing生态系统中，span!宏是构建分布式追踪系统的核心工具。它允许你创建一个**操作范围**（span），用于记录程序执行过程中的上下文信息和时间消耗。后续日志中的确包含了添加的用户信息，但是存在用户信息重复添加的问题，我感觉应该是span的作用范围和生命周期导致的。

## 最终解决方案
最后的解决方案，还是通过layer中间件解决的。使用的是tower-http的TraceLayer中间件，在TraceLayer的make_span_with方法中，将request header中的token信息提取解密后，通过span!宏进行记录，后续打印的日志中就包含了用户信息。

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
