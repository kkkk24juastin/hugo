# 我的Hugo博客

这是一个使用 Hugo 静态网站生成器搭建的个人博客，采用 GitHub Actions 自动编译生成静态文件。

## 📁 项目结构

```
.
├── .github/
│   └── workflows/
│       └── hugo.yml          # GitHub Actions 工作流配置
├── archetypes/
│   └── default.md            # 文章模板
├── content/
│   ├── posts/                # 博客文章目录
│   │   └── hello-world.md    # 示例文章
│   ├── about.md              # 关于页面
│   └── archives.md           # 归档页面
├── themes/
│   └── PaperMod/             # 主题（git submodule）
├── .gitignore
├── hugo.toml                 # Hugo 配置文件
└── README.md
```

## 🚀 快速开始

### 1. 初始化 Git 仓库并添加主题

```bash
# 初始化 Git 仓库
git init

# 添加 PaperMod 主题作为 submodule
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# 提交更改
git add .
git commit -m "Initial commit: Hugo blog setup"
```

### 2. 推送到 GitHub

```bash
# 添加远程仓库（替换为你的仓库地址）
git remote add origin https://github.com/yourusername/hugo-blog.git

# 推送到 main 分支
git branch -M main
git push -u origin main
```

### 3. 自动编译

推送到 `main` 分支后，GitHub Actions 会自动：
1. 编译生成静态文件
2. 将静态文件保存为 Artifact（可在 Actions 页面下载）
3. 将静态文件推送到 `gh-pages` 分支

## 🖥️ 在服务器上部署

### 方法一：拉取 gh-pages 分支

```bash
# 克隆 gh-pages 分支到服务器
git clone --branch gh-pages --single-branch https://github.com/yourusername/hugo-blog.git /var/www/blog

# 后续更新
cd /var/www/blog
git pull
```

### 方法二：使用 Webhook 自动更新

在服务器上设置 Webhook，当 `gh-pages` 分支更新时自动拉取最新文件。

### 方法三：下载 Artifact

在 GitHub Actions 页面下载最新的 `hugo-site` artifact，解压到服务器。

## ✍️ 写新文章

### 使用 Hugo 命令创建（推荐）

```bash
# 如果本地安装了 Hugo
hugo new posts/my-new-post.md
```

### 手动创建

在 `content/posts/` 目录下创建新的 Markdown 文件：

```markdown
---
title: "文章标题"
date: 2024-01-01
draft: false
tags: ["标签1", "标签2"]
categories: ["分类"]
description: "文章描述"
---

文章内容...
```

## 🔧 自定义配置

编辑 `hugo.toml` 文件来自定义你的博客：

- `baseURL`: 修改为你的博客实际访问地址
- `title`: 博客标题
- `params.author`: 作者名称
- `params.description`: 博客描述
- `params.socialIcons`: 社交媒体链接

## 📦 本地预览（可选）

如果你想在本地预览博客，需要安装 Hugo：

```bash
# macOS
brew install hugo

# Ubuntu/Debian
sudo apt install hugo

# 或者从 GitHub Releases 下载
# https://github.com/gohugoio/hugo/releases

# 启动本地服务器
hugo server -D
```

然后访问 http://localhost:1313 预览博客。

## 🎨 更换主题

1. 删除当前主题 submodule
2. 添加新主题作为 submodule
3. 更新 `hugo.toml` 中的 `theme` 配置
4. 根据新主题的文档调整配置

## 📝 工作流程

1. **本地**：只存储文章源文件（`content/`）和主题配置
2. **GitHub Actions**：自动编译生成静态文件
3. **gh-pages 分支**：存储编译后的静态文件
4. **服务器**：拉取 `gh-pages` 分支的静态文件

## 📄 许可证

文章内容采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。
