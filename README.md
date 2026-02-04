# kkkk24 的博客

这是一个使用 [Hugo](https://gohugo.io/) 静态网站生成器和 [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 主题搭建的个人博客。

🌐 **博客地址**: https://blog.kkkk24juastin.asia/

采用 GitHub Actions 自动编译生成静态文件，供服务器拉取部署。

## 📁 项目结构

```
.
├── .github/
│   └── workflows/
│       └── hugo.yml              # GitHub Actions 工作流配置
├── archetypes/
│   └── default.md                # 文章模板
├── assets/
│   └── css/
│       └── extended/
│           └── custom.css        # 自定义 CSS
├── content/
│   ├── posts/                    # 博客文章目录
│   │   └── hello-world.md        # 示例文章
│   ├── about.md                  # 关于页面
│   ├── archives.md               # 归档页面
│   └── search.md                 # 搜索页面
├── layouts/
│   └── partials/
│       ├── comments.html         # 评论系统配置
│       ├── extend_head.html      # 自定义 Head
│       └── extend_footer.html    # 自定义 Footer
├── static/                       # 静态资源 (图片、favicon等)
├── themes/
│   └── PaperMod/                 # 主题 (git submodule)
├── .gitignore
├── hugo.toml                     # Hugo 配置文件
└── README.md
```

## 🚀 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/kkkk24/hugo.git
cd hugo

# 初始化 submodule（获取主题）
git submodule update --init --recursive
```

### 2. 推送更改

```bash
git add .
git commit -m "Update content"
git push
```

### 3. 自动编译

推送到 `main` 分支后，GitHub Actions 会自动：
1. 使用 Hugo 0.155.2 编译生成静态文件
2. 将静态文件保存为 Artifact（可在 Actions 页面下载）
3. 将静态文件推送到 `gh-pages` 分支

## 🖥️ 在服务器上部署

### 方法一：拉取 gh-pages 分支

```bash
# 首次克隆
git clone --branch gh-pages --single-branch https://github.com/kkkk24/hugo.git /var/www/blog

# 后续更新
cd /var/www/blog && git pull
```

### 方法二：使用 Webhook 自动更新

配置 GitHub Webhook，当 `gh-pages` 分支更新时自动触发服务器拉取。

## ✍️ 写新文章

### 使用 Hugo 命令创建（推荐）

```bash
# 如果本地安装了 Hugo
hugo new posts/my-new-post.md
```

### 手动创建

在 `content/posts/` 目录下创建新的 Markdown 文件，参考 `archetypes/default.md` 模板。

### 文章 Front Matter 说明

```yaml
---
title: "文章标题"
date: 2024-01-01
draft: false                    # 是否为草稿
weight: 1                       # 置顶权重
tags: ["标签1", "标签2"]
categories: ["分类"]
author: ["kkkk24"]
description: "文章描述"
showToc: true                   # 显示目录
TocOpen: false                  # 目录默认展开
comments: true                  # 显示评论
cover:
    image: "cover.jpg"          # 封面图片
    alt: "封面描述"
    caption: "图片说明"
---
```

## 🎨 自定义

### 自定义 CSS

编辑 `assets/css/extended/custom.css` 添加自定义样式。

### 自定义 Head/Footer

- `layouts/partials/extend_head.html`: 添加自定义 meta 标签、CSS、字体等
- `layouts/partials/extend_footer.html`: 添加自定义 JavaScript、统计代码等

### 添加评论系统

编辑 `layouts/partials/comments.html`，取消注释并配置你选择的评论系统：
- **Giscus** (推荐): 基于 GitHub Discussions
- **Utterances**: 基于 GitHub Issues
- **Disqus**: 第三方评论系统

### 添加 Favicon

将以下文件放入 `static/` 目录：
- `favicon.ico`
- `favicon-16x16.png`
- `favicon-32x32.png`
- `apple-touch-icon.png`
- `safari-pinned-tab.svg`

可使用 [Favicon.io](https://favicon.io) 生成。

## 📦 本地预览（可选）

```bash
# 安装 Hugo (macOS)
brew install hugo

# 安装 Hugo (Ubuntu/Debian)
sudo apt install hugo

# 或从 GitHub Releases 下载最新版本
# https://github.com/gohugoio/hugo/releases

# 启动本地服务器
hugo server -D

# 访问 http://localhost:1313
```

## 🔑 快捷键

PaperMod 主题支持以下快捷键：

| 快捷键 | 功能 |
|--------|------|
| `c` | 展开/收起目录 |
| `g` | 回到顶部 |
| `h` | 回到首页 |
| `t` | 切换主题 |
| `/` | 跳转到搜索页 |

## 📝 工作流程

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   本地编辑       │     │  GitHub Actions  │     │    服务器        │
│   (Markdown)    │ --> │  (编译生成静态)   │ --> │  (托管静态文件)  │
└─────────────────┘     └──────────────────┘     └─────────────────┘
        │                        │                       │
        │                        ▼                       │
        │               ┌──────────────────┐            │
        │               │   gh-pages 分支   │◄───────────┘
        │               │   (静态文件)      │    git pull
        │               └──────────────────┘
        │                        │
        ▼                        ▼
  git push main          Artifact 下载
```

## 📚 参考文档

- [Hugo 官方文档](https://gohugo.io/documentation/)
- [PaperMod Wiki](https://github.com/adityatelange/hugo-PaperMod/wiki)
  - [安装指南](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation)
  - [功能特性](https://github.com/adityatelange/hugo-PaperMod/wiki/Features)
  - [常见问题](https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs)
  - [变量说明](https://github.com/adityatelange/hugo-PaperMod/wiki/Variables)
  - [图标列表](https://github.com/adityatelange/hugo-PaperMod/wiki/Icons)

## 📄 许可证

文章内容采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。
