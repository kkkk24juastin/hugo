---
title: "Hugo 博客速度优化实践 V2 - 深度性能分析"
date: 2026-02-06
draft: false
description: "通过实际性能测试，发现博客的核心瓶颈并针对性优化"
summary: "基于真实性能数据的博客优化方案：TTFB 从 2.28s 优化到 200ms 以下"
tags: ["Hugo", "性能优化", "Cloudflare", "CDN"]
categories: ["技术笔记"]
author: "kkkk24"
showToc: true
TocOpen: true
---

## 性能测试结果

使用 curl 对博客进行了性能测试：

```bash
curl -s -o /dev/null -w "HTTP Status: %{http_code}\nDNS Lookup: %{time_namelookup}s\nConnect: %{time_connect}s\nTTFB: %{time_starttransfer}s\nTotal Time: %{time_total}s\nSize: %{size_download} bytes\n" https://blog.kkkk24juastin.asia/
```

**测试结果：**

| 指标 | 当前值 | 目标值 | 状态 |
|------|--------|--------|------|
| DNS 解析 | 25ms | < 50ms | ✅ 良好 |
| TCP 连接 | 25ms | < 100ms | ✅ 良好 |
| TTFB | **2.28s** | < 200ms | ❌ 需优化 |
| 总耗时 | 2.29s | < 500ms | ❌ 需优化 |
| HTML 大小 | 16KB | - | ✅ 良好 |

**HTTP 响应头分析：**

```
cf-cache-status: DYNAMIC  ← 问题所在！
cache-control: max-age=3600
server: cloudflare
```

## 发现的核心问题

### 1. Cloudflare 未缓存 HTML (最关键)

`cf-cache-status: DYNAMIC` 表示 Cloudflare 没有缓存这个页面，每次请求都要回源到服务器。

对于静态博客来说，HTML 应该被缓存，这会让 TTFB 从 2+ 秒降到 < 50ms！

### 2. 缺少 Brotli/Gzip 压缩确认

响应头中没有显示 `Content-Encoding: br` 或 `gzip`。

## 优化方案

### 方案一：Cloudflare 页面规则（推荐）

在 Cloudflare Dashboard 中创建页面规则：

1. 登录 Cloudflare Dashboard
2. 选择你的域名 `kkkk24juastin.asia`
3. 进入 **Rules** → **Page Rules**
4. 创建新规则：

```
URL: blog.kkkk24juastin.asia/*
设置:
  - Cache Level: Cache Everything
  - Edge Cache TTL: 1 month
  - Browser Cache TTL: 1 day
```

### 方案二：Cloudflare Cache Rules（新版）

如果使用新版 Cloudflare 规则：

1. 进入 **Caching** → **Cache Rules**
2. 创建规则：

```
匹配: 主机名 equals "blog.kkkk24juastin.asia"
操作:
  - 符合缓存条件: 是
  - 边缘 TTL: 无视源站，缓存 1 个月
  - 浏览器 TTL: 无视源站，缓存 1 天
```

### 方案三：自定义 _headers 文件

在 `static/` 目录下创建 `_headers` 文件：

```
/*
  Cache-Control: public, max-age=31536000

/*.html
  Cache-Control: public, max-age=86400

/index.json
  Cache-Control: public, max-age=3600
```

### 方案四：开启 Brotli 压缩

在 Cloudflare Dashboard：

1. 进入 **Speed** → **Optimization**
2. 确保 **Brotli** 已开启
3. 确保 **Auto Minify** 已开启（JS, CSS, HTML）

## 代码层面优化

### 1. 添加资源预加载提示

在 `layouts/partials/extend_head.html` 中添加：

```html
{{/* 预加载关键资源 */}}
<link rel="preload" href="{{ (resources.Get "css/common/header.css").RelPermalink }}" as="style">
```

### 2. 内联关键 CSS

对于首屏渲染需要的 CSS，可以考虑内联。

### 3. 优化图片加载

确保所有非首屏图片都使用 `loading="lazy"`（已实现）。

### 4. 添加 fetchpriority 提示

```html
<link rel="preload" href="..." as="style" fetchpriority="high">
```

## 优化后预期效果

| 指标 | 优化前 | 优化后预期 |
|------|--------|------------|
| TTFB (首次) | 2.28s | < 200ms |
| TTFB (缓存命中) | 2.28s | < 50ms |
| FCP | ~2.5s | < 1s |
| LCP | ~3s | < 1.5s |

## 验证方法

优化后运行：

```bash
# 第一次请求（预热缓存）
curl -s -o /dev/null -w "TTFB: %{time_starttransfer}s\n" https://blog.kkkk24juastin.asia/

# 第二次请求（验证缓存）
curl -s -o /dev/null -w "TTFB: %{time_starttransfer}s\n" https://blog.kkkk24juastin.asia/

# 检查缓存状态
curl -sI https://blog.kkkk24juastin.asia/ | grep cf-cache-status
# 期望看到: cf-cache-status: HIT
```

## 总结

对于静态博客来说，最大的优化来自于 CDN 缓存。代码层面的优化（minify、懒加载等）已经做得很好了，但如果 Cloudflare 不缓存页面，每次都回源到服务器，前端优化的效果会被抵消。

**优先级排序：**

1. 🔥 **Cloudflare 缓存规则** - 预期提升 90%+ 
2. 🔶 **Brotli 压缩** - 预期提升 20-40%
3. 🔶 **资源预加载** - 预期提升 10-20%
4. 🔷 **关键 CSS 内联** - 预期提升 5-10%
