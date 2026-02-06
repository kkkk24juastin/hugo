---
title: "博客加载太慢？我是这样排查和解决的"
date: 2026-02-06
draft: false
description: "发现博客首页加载要 2 秒多，一顿排查之后找到了罪魁祸首"
summary: "从 2.28s 优化到 50ms，原来问题出在 Cloudflare 没缓存 HTML"
tags: ["性能优化", "Cloudflare", "CDN"]
categories: ["折腾记录"]
author: "kkkk24"
showToc: true
TocOpen: true
---

## 起因

前两天觉得博客打开有点慢，但也没太在意。今天闲着没事，拿 curl 测了一下，好家伙，首字节时间（TTFB）居然要 2.28 秒：

```bash
curl -s -o /dev/null -w "TTFB: %{time_starttransfer}s\n" https://blog.kkkk24juastin.asia/

# 输出：TTFB: 2.280466s
```

这也太离谱了。明明挂着 Cloudflare CDN，怎么还这么慢？

## 排查过程

先看看响应头：

```bash
curl -sI https://blog.kkkk24juastin.asia/ | grep -i cache

# 输出：
# cf-cache-status: DYNAMIC
# cache-control: max-age=3600
```

看到 `cf-cache-status: DYNAMIC` 我就懂了。

这玩意的意思是 Cloudflare 压根没缓存这个页面，每次访问都要回源到我的服务器。而我的服务器在国外，网络延迟加上 TLS 握手，2 秒多其实不算意外。

## 问题出在哪

Cloudflare 有个"特性"：**默认不缓存 HTML 页面**。

它的逻辑是 HTML 通常是动态生成的，不适合缓存。但对于静态博客来说，HTML 就是静态文件啊，为啥不能缓存？

## 解决方案

需要手动加一条 Cache Rule，告诉 Cloudflare：这个域名下的 HTML 也给我缓存上。

操作步骤：

1. 登录 Cloudflare Dashboard
2. 选你的域名
3. 进 **Caching** → **Cache Rules**
4. 创建规则：

```
匹配条件: 主机名 = blog.kkkk24juastin.asia
缓存设置: 
  - 符合缓存条件: 是
  - 边缘 TTL: 1 个月
  - 浏览器 TTL: 1 天
```

保存，等几分钟生效。

## 效果验证

再跑一遍：

```bash
# 第一次，预热缓存
curl -s -o /dev/null -w "TTFB: %{time_starttransfer}s\n" https://blog.kkkk24juastin.asia/
# TTFB: 1.1s（回源了）

# 第二次，缓存命中
curl -s -o /dev/null -w "TTFB: %{time_starttransfer}s\n" https://blog.kkkk24juastin.asia/
# TTFB: 0.045s

# 确认缓存状态
curl -sI https://blog.kkkk24juastin.asia/ | grep cf-cache-status
# cf-cache-status: HIT
```

从 2.28s 降到 45ms，提升了 98%。

舒服了。

## 还做了啥

既然都折腾了，顺手做了几个小优化：

### 1. 确认 Brotli 压缩开了

在 Cloudflare 的 Speed → Optimization 里，确保 Brotli 是开着的。比 Gzip 压缩率更高。

### 2. 加了资源预加载

在 `extend_head.html` 里加了 DNS 预解析：

```html
<link rel="dns-prefetch" href="//fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.googleapis.com" crossorigin>
```

### 3. 构建时自动清缓存

在 GitHub Actions 里加了一步，每次部署完自动清 Cloudflare 缓存：

```yaml
- name: 清缓存
  run: |
    curl -X POST "https://api.cloudflare.com/client/v4/zones/$CF_ZONE_ID/purge_cache" \
      -H "Authorization: Bearer $CF_API_TOKEN" \
      --data '{"purge_everything":true}'
```

这样每次更新文章，CDN 会立刻拉取最新内容。

## 总结

对于静态博客来说，优化的重点不在代码层面（Hugo 已经够快了），而是**让 CDN 真正发挥作用**。

如果你的博客也挂着 Cloudflare 但还是慢，先检查一下 `cf-cache-status` 是不是 `HIT`。不是的话，多半是缓存规则没配好。

就这样，收工。
