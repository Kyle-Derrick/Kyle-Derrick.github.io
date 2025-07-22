---
title: '开源库搜寻记录'
author: ['kyle']
date: '2025-07-22T23:11:48+08:00'
tags:
- Rust
- 开源

keywords:
- Rust
- 开源
---

> 主要用于寻找第三方库后做备忘的零散记录，常用库此处不记录

# Rust

deno: 类似nodejs
nom: 解析库（包括但不限于语法）
egui、Tauri、Iced: GUI框架：https://rustcc.cn/article?id=8f6c816e-790e-44ff-8c80-1a0b8154fb7d

pingora 类似nginx

### 加密库
> aes等加密以及ed25519等签名校验常用ring
rustls、rust-openssl、[ring](https://github.com/briansmith/ring)

### JS相关
https://github.com/yuankunzhang/charming : 通过deno_core调用echarts生成图表
https://github.com/rscarson/rustyscript : deno_core的上层封装

boa_engine : 一个实验性Js引擎，rust实现

# Java
javalin: 非常轻量级的web框架（java或kotlin）
vertx: 响应式 网络框架集 包括但不限于web
quarkus: 全栈Kubernetes原生Java框架，专为GraalVM和HotSpot优化，底层使用的反应式框架之一是vertx

# 浏览器自动化
## 一、Playwright vs Puppeteer 核心对比

**对比维度** | **Puppeteer** | **Playwright**
---  | --- | ---
**浏览器支持** | 主攻 Chromium，有限支持 Firefox（功能不全） | 原生支持 Chromium、Firefox、WebKit（Safari）
**API 设计** | 需手动等待元素状态（如 waitForSelector） | 自动等待元素可交互（如点击前检查可见性）
**性能表现** | 冷启动约 1.2 秒，内存占用较低 | 冷启动约 0.8 秒（连接池优化），支持并行上下文隔离
**高级功能** | - PDF 生成质量更高<br/>- 截图速度快 15-20% | - 移动设备模拟（精确视口/触摸事件）<br/>- 视频录制、网络请求拦截<br/>- 跨 iframe 操作
**多语言支持** | Node.js 为主，社区提供非官方 Python 库 | 官方支持：JavaScript/TypeScript、Python、Java、C#
**适用场景** | - 仅需覆盖 Chromium 系浏览器 | - 轻量级部署或高质量 PDF 生成<br/>- 跨浏览器测试（含 Safari）<br/>- 复杂自动化（移动端模拟、视频录制）<br/>- 多语言团队协作


**选型建议**：

* **选 Puppeteer**：项目深度绑定 Chromium 生态，追求轻量部署或高质量 PDF/截图生成。
* **选 Playwright**：需覆盖 Safari/Firefox、实现复杂自动化（如移动端测试），或团队使用多语言开发。
