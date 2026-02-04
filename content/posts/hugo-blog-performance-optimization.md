---
title: "Hugo 博客加载速度优化实践"
date: 2026-02-04
draft: false
description: "记录优化 Hugo + PaperMod 博客加载速度的完整过程，包括构建配置、图片懒加载、资源压缩等多个方面"
summary: "博客上线后发现加载速度不太理想，于是花了点时间研究优化方案。这篇文章记录了从 Hugo 配置到前端样式的完整优化过程。"
tags: ["Hugo", "性能优化", "前端", "博客"]
categories: ["技术笔记"]
author: "kkkk24"
showToc: true
TocOpen: false
---

## 发现问题

博客搭建完成后，访问时总觉得有点卡。打开 Chrome DevTools 看了下 Network 面板，发现几个问题：

1. 图片没有做懒加载，首屏就加载了所有图片
2. HTML/CSS/JS 文件可以进一步压缩
3. 没有利用好浏览器缓存
4. 外部资源（如字体）请求有延迟

既然发现了问题，那就一个个解决。

## Hugo 构建优化

### 深度压缩配置

Hugo 自带 minify 功能，但默认配置比较保守。在 `hugo.toml` 中可以配置更激进的压缩选项：

```toml
[minify]
  disableXML = true
  minifyOutput = true

  [minify.tdewolff]
    [minify.tdewolff.css]
      keepCSS2 = false
      precision = 0
    
    [minify.tdewolff.html]
      keepDocumentTags = true
      keepEndTags = true
      keepQuotes = false
      keepWhitespace = false
    
    [minify.tdewolff.js]
      keepVarNames = false
      precision = 0
    
    [minify.tdewolff.json]
      keepNumbers = false
      precision = 0
    
    [minify.tdewolff.svg]
      keepComments = false
      precision = 0
```

这个配置使用了 [tdewolff/minify](https://github.com/tdewolff/minify) 库的参数，可以对各类资源做深度压缩：

- **CSS**: 移除不必要的空格、简化数值精度
- **HTML**: 移除空白、可选引号
- **JS**: 压缩变量名
- **SVG**: 移除注释、简化数值

### 构建缓存

重复构建时，很多资源是不变的，可以利用缓存加速：

```toml
[build]
  writeStats = true

[caches]
  [caches.assets]
    dir = ":resourceDir/_gen"
    maxAge = "720h"  # 30 天
  
  [caches.getcsv]
    dir = ":cacheDir/:project"
    maxAge = "4h"
  
  [caches.getjson]
    dir = ":cacheDir/:project"
    maxAge = "4h"
  
  [caches.getresource]
    dir = ":cacheDir/:project"
    maxAge = "4h"
  
  [caches.images]
    dir = ":resourceDir/_gen"
    maxAge = "720h"
```

`writeStats = true` 会生成一个包含所有使用过的类名的文件，可以配合 PurgeCSS 做 tree-shaking（虽然 PaperMod 主题本身已经很精简了）。

### 图片处理优化

Hugo 的图片处理功能很强大，合理配置可以减小图片体积：

```toml
[imaging]
  quality = 80
  resampleFilter = "Lanczos"
  hint = "photo"
  anchor = "Smart"
  bgColor = "#ffffff"
  
  [imaging.exif]
    disableDate = false
    disableLatLong = true  # 保护隐私
```

- `quality = 80`：图片质量设为 80%，肉眼几乎看不出差别，但体积能减小不少
- `resampleFilter = "Lanczos"`：高质量重采样算法，适合缩放照片
- `anchor = "Smart"`：智能裁剪，自动识别图片主体
- `disableLatLong = true`：移除 EXIF 中的位置信息，保护隐私

## 前端优化

### DNS 预解析和预连接

浏览器解析域名需要时间，可以用 `dns-prefetch` 提前解析外部域名：

```html
<!-- 放在 layouts/partials/extend_head.html -->
<link rel="dns-prefetch" href="//fonts.googleapis.com">
<link rel="dns-prefetch" href="//fonts.gstatic.com">
<link rel="dns-prefetch" href="//cdn.jsdelivr.net">

<link rel="preconnect" href="https://fonts.googleapis.com" crossorigin>
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
```

`preconnect` 比 `dns-prefetch` 更进一步，会提前建立完整的 TCP 连接。

### 图片懒加载

现代浏览器都支持原生的 `loading="lazy"` 属性，不需要额外的 JS 库：

```javascript
document.addEventListener('DOMContentLoaded', function() {
  function addLoadedClass(img) {
    if (img.complete) {
      img.classList.add('loaded');
    } else {
      img.addEventListener('load', function() {
        img.classList.add('loaded');
      });
    }
  }
  
  document.querySelectorAll('.post-content img').forEach(function(img) {
    if (!img.hasAttribute('loading')) {
      img.setAttribute('loading', 'lazy');
      img.setAttribute('decoding', 'async');
    }
    addLoadedClass(img);
  });
});
```

这段代码做了两件事：

1. 给所有文章图片添加 `loading="lazy"` 和 `decoding="async"` 属性
2. 图片加载完成后添加 `loaded` 类，配合 CSS 做渐显效果

### 图片渐显效果

配合上面的 JS，在 CSS 中实现平滑的渐显效果：

```css
/* assets/css/extended/custom.css */

.post-content img,
.entry-cover img {
  background-color: var(--code-bg);
  transition: opacity 0.3s ease-in-out;
}

.post-content img[loading="lazy"],
.entry-cover img[loading="lazy"] {
  opacity: 0;
}

.post-content img.loaded,
.entry-cover img.loaded {
  opacity: 1;
}
```

这样图片在加载时会显示一个灰色占位背景，加载完成后平滑显示。

### 减少布局偏移

图片加载时如果没有预留空间，会导致页面布局跳动（CLS 指标），用户体验很差：

```css
.post-content img {
  aspect-ratio: attr(width) / attr(height);
  height: auto;
  max-width: 100%;
}
```

如果图片设置了 width 和 height 属性，浏览器可以提前计算出图片的宽高比，预留好空间。

### 尊重用户偏好

有些用户可能对动画敏感，可以检测并禁用过渡效果：

```css
@media (prefers-reduced-motion: no-preference) {
  html {
    scroll-behavior: smooth;
  }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

## 优化效果

做完这些优化后，再看 Lighthouse 跑分：

| 指标 | 优化前 | 优化后 |
|------|--------|--------|
| 首次内容绘制 (FCP) | ~1.5s | ~0.8s |
| 最大内容绘制 (LCP) | ~2.5s | ~1.5s |
| 累积布局偏移 (CLS) | 0.15 | < 0.05 |

当然，实际效果还取决于服务器性能和用户网络。

## 更多优化思路

这次只做了 Hugo 本身的优化，还有一些可以继续优化的方向：

1. **CDN 加速**：把静态资源放到 CDN 上，全球访问都很快
2. **Gzip/Brotli 压缩**：服务器开启压缩，进一步减小传输体积
3. **HTTP/2**：启用 HTTP/2 多路复用，并行加载资源
4. **字体优化**：使用 `font-display: swap` 避免字体阻塞渲染
5. **图片格式**：使用 WebP/AVIF 等现代格式

后续有空再继续折腾。

## 参考资料

- [Hugo 官方文档 - Configure Minify](https://gohugo.io/getting-started/configuration/#configure-minify)
- [Hugo 官方文档 - Image Processing](https://gohugo.io/content-management/image-processing/)
- [Web.dev - Browser-level image lazy loading](https://web.dev/browser-level-image-lazy-loading/)
- [Web.dev - Optimize Cumulative Layout Shift](https://web.dev/optimize-cls/)
