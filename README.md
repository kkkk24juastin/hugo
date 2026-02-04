# 我的Hugo博客

这是一个使用 Hugo 静态网站生成器搭建的个人博客，采用 GitHub Actions 自动构建并部署到 GitHub Pages。

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
git remote add origin https://github.com/yourusername/yourusername.github.io.git

# 推送到 main 分支
git branch -M main
git push -u origin main
```

### 3. 配置 GitHub Pages

1. 在 GitHub 仓库中，进入 **Settings** > **Pages**
2. 在 **Source** 下选择 **GitHub Actions**
3. 等待 Actions 完成构建，你的博客就会自动部署

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

- `baseURL`: 修改为你的博客地址
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

## 📝 注意事项

- 本仓库只存储源文件（文章、主题配置等）
- 静态文件由 GitHub Actions 自动生成
- 推送到 `main` 分支会自动触发部署
- `public/` 目录已被 `.gitignore` 忽略

## 📄 许可证

文章内容采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。
