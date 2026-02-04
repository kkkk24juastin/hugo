---
title: "Hello World - 我的第一篇博客文章"
date: 2026-02-04
lastmod: 2026-02-04
draft: false
author: ["kkkk24"]
tags: ["Hugo", "博客", "PaperMod"]
categories: ["入门"]
description: "这是我使用Hugo搭建博客后的第一篇文章，介绍了博客的架构和特性"
weight: 1
slug: "hello-world"
comments: true
showToc: true
TocOpen: true
hidemeta: false
disableShare: false
showbreadcrumbs: true
cover:
    image: ""
    caption: ""
    alt: ""
    relative: false
---

## 欢迎来到我的博客！

这是使用 [Hugo](https://gohugo.io/) 框架和 [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 主题搭建的个人博客。

博客地址：https://blog.kkkk24juastin.asia/

### 为什么选择Hugo？

Hugo 是目前最流行的静态网站生成器之一，具有以下优势：

- **速度快** - Hugo 是用 Go 语言编写的，构建速度极快
- **简单易用** - 使用 Markdown 编写内容，专注于写作
- **主题丰富** - 有大量优秀的主题可供选择
- **部署简单** - 生成静态文件，可以部署到任何 Web 服务器

### 我的博客架构

本博客采用以下架构：

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   本地编辑       │     │  GitHub Actions  │     │    服务器        │
│   (Markdown)    │ --> │  (编译生成静态)   │ --> │  (托管静态文件)  │
└─────────────────┘     └──────────────────┘     └─────────────────┘
```

1. **本地**: 只存储文章和主题源文件
2. **GitHub Actions**: 自动编译生成静态文件并推送到 `gh-pages` 分支
3. **服务器**: 从 `gh-pages` 分支拉取静态文件进行托管

### PaperMod 主题特性

PaperMod 是一个快速、简洁、响应式的 Hugo 主题，支持：

| 特性 | 描述 |
|------|------|
| 🌓 深色/浅色模式 | 自动适应系统主题或手动切换 |
| 🔍 全文搜索 | 基于 Fuse.js 的快速搜索 |
| 📱 响应式设计 | 完美适配各种设备 |
| 📑 目录导航 | 自动生成文章目录 |
| 🏷️ 标签和分类 | 方便文章组织和查找 |
| 📊 阅读时间估算 | 显示预计阅读时间 |

### 代码示例

Hugo 支持多种语言的语法高亮：

```python
def hello_world():
    """一个简单的 Python 函数"""
    print("Hello, Hugo!")
    return True

if __name__ == "__main__":
    hello_world()
```

```javascript
// JavaScript 示例
const greet = (name) => {
    console.log(`Hello, ${name}!`);
};

greet('Hugo');
```

### Markdown 扩展

PaperMod 支持丰富的 Markdown 扩展语法：

> 这是一段引用文字
> 
> — 某位名人

---

**粗体文本** 和 *斜体文本* 以及 ~~删除线~~

- 无序列表项 1
- 无序列表项 2
  - 嵌套列表项

1. 有序列表项 1
2. 有序列表项 2

### 下一步计划

- [x] 搭建博客基础框架
- [x] 配置 GitHub Actions 自动构建
- [ ] 定制主题样式
- [ ] 添加评论系统
- [ ] 配置 SEO 优化
- [ ] 添加更多内容

感谢阅读！🎉

如果这篇文章对你有帮助，欢迎分享给更多人。
