---
title: "给 Hexo 博客加个 Live2D 看板娘"
date: 2026-02-03T01:29:00+08:00
lastmod: 2026-02-03T01:29:00+08:00
draft: false
author: ["kkkk24"]
tags: ["Hexo", "Live2D", "博客美化", "前端"]
categories: ["技术教程"]
description: "手把手教你给 Hexo 博客添加一个萌萌的 Live2D 看板娘，让博客不再单调"

showToc: true
TocOpen: true
hidemeta: false
comments: true
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true

cover:
    image: ""
    alt: "Live2D 看板娘"
    caption: ""
    relative: false
    hidden: false
---

之前逛别人博客的时候，经常看到左下角有个会动的小人，眼睛还会跟着鼠标跑，感觉挺有意思的。研究了一下发现这玩意叫 Live2D，于是决定给自己的 Hexo 博客也整一个。

折腾了一晚上，踩了不少坑，这里记录一下实现过程，希望能帮到有同样需求的朋友。

## 先看效果

加完之后，博客左下角会多一个二次元小姐姐（或者其他角色），她能：

- 眼睛跟着你鼠标动（有点魔性）
- 随机显示一言（hitokoto）
- 换装、换模型
- 截图保存
- 还能玩个打砖块小游戏...

说实话，虽然没啥实际用处，但看着就是舒服 😂

## 最简单的方式

如果你只是想快速体验一下，一行代码就够了。

在主题的 `layout/layout.ejs`（或者类似的布局文件）里加上：

```html
<script src="https://fastly.jsdelivr.net/npm/live2d-widgets@1.0.0/dist/autoload.js"></script>
```

放在 `</body>` 之前就行。刷新页面，看板娘应该就出来了。

不过这种方式没法自定义配置，想要更多控制就得用下面的方法。

## 完整配置版

### 第一步：创建看板娘组件

在主题目录的 `layout/_partial/` 下面新建一个 `live2d.ejs` 文件：

```ejs
<%# 看板娘组件 %>
<% if (theme.live2d && theme.live2d.enable) { %>
  <%
    // 读取配置，给个默认值兜底
    const cdnPath = (theme.live2d.cdn && theme.live2d.cdn.url) 
      ? theme.live2d.cdn.url 
      : 'https://fastly.jsdelivr.net/npm/live2d-widgets@1.0.0/dist/';
    const modelId = (theme.live2d.model && theme.live2d.model.id !== undefined) 
      ? theme.live2d.model.id 
      : 0;
    const modelCdnPath = (theme.live2d.model && theme.live2d.model.cdnPath) 
      ? theme.live2d.model.cdnPath 
      : 'https://fastly.jsdelivr.net/gh/fghrsh/live2d_api/';
    const toolItems = (theme.live2d.tools && theme.live2d.tools.items) 
      ? theme.live2d.tools.items 
      : ['hitokoto', 'asteroids', 'switch-model', 'switch-texture', 'photo', 'info', 'quit'];
    const drag = theme.live2d.drag !== undefined ? theme.live2d.drag : false;
    const logLevel = theme.live2d.logLevel || 'warn';
  %>

  <script>
    const live2d_path = '<%= cdnPath %>';
    
    // 动态加载资源的函数
    function loadExternalResource(url, type) {
      return new Promise((resolve, reject) => {
        let tag;
        if (type === 'css') {
          tag = document.createElement('link');
          tag.rel = 'stylesheet';
          tag.href = url;
        } else if (type === 'js') {
          tag = document.createElement('script');
          tag.type = 'module';
          tag.src = url;
        }
        if (tag) {
          tag.onload = () => resolve(url);
          tag.onerror = () => reject(url);
          document.head.appendChild(tag);
        }
      });
    }
    
    (async () => {
      // 手机上就别加载了，太占地方
      if (screen.width < 768) return;
      
      // 处理图片跨域问题（这个坑踩了好久）
      const OriginalImage = window.Image;
      window.Image = function(...args) {
        const img = new OriginalImage(...args);
        img.crossOrigin = "anonymous";
        return img;
      };
      window.Image.prototype = OriginalImage.prototype;
      
      // 加载样式和脚本
      await Promise.all([
        loadExternalResource(live2d_path + 'waifu.css', 'css'),
        loadExternalResource(live2d_path + 'waifu-tips.js', 'js')
      ]);
      
      // 初始化
      initWidget({
        waifuPath: live2d_path + 'waifu-tips.json',
        cdnPath: '<%= modelCdnPath %>',
        cubism2Path: live2d_path + 'live2d.min.js',
        cubism5Path: 'https://cubism.live2d.com/sdk-web/cubismcore/live2dcubismcore.min.js',
        modelId: <%= modelId %>,
        tools: <%- JSON.stringify(toolItems) %>,
        drag: <%= drag %>,
        logLevel: '<%= logLevel %>'
      });
    })();
  </script>
<% } %>
```

### 第二步：引入组件

在 `layout/layout.ejs` 里引入（放在 `</body>` 前面）：

```ejs
<%- partial('_partial/live2d') %>
```

### 第三步：配置主题

在主题的 `_config.yml` 里添加配置：

```yaml
# Live2D 看板娘
live2d:
  enable: true
  model:
    id: 0                    # 默认模型，可以通过按钮切换
    cdnPath: https://fastly.jsdelivr.net/gh/fghrsh/live2d_api/
  tools:
    items:
      - hitokoto             # 一言
      - asteroids            # 小游戏
      - switch-model         # 切换模型
      - switch-texture       # 切换皮肤
      - photo                # 拍照
      - info                 # 关于
      - quit                 # 关闭
  drag: false                # 能不能拖动，我觉得固定在角落挺好
  logLevel: warn
  cdn:
    url: https://fastly.jsdelivr.net/npm/live2d-widgets@1.0.0/dist/
```

## 配置项说明

简单说一下各个选项：

| 配置项 | 说明 |
|-------|------|
| `enable` | 开关，不想要了直接改成 false |
| `model.id` | 默认显示哪个模型，0 是第一个 |
| `model.cdnPath` | 模型资源地址，用默认的就行 |
| `tools.items` | 工具栏显示哪些按钮 |
| `drag` | 能不能拖动看板娘 |
| `cdn.url` | 组件资源地址 |

## 遇到的几个坑

### 1. 看板娘变形了

一开始我在自己的 CSS 里给看板娘的容器设置了宽高，结果模型比例全乱了。后来发现要让它自己加载 `waifu.css`，不要手动设置尺寸。

### 2. 图片跨域报错

控制台一堆跨域错误，截图功能也用不了。解决方法是在加载看板娘之前，给 Image 对象设置 `crossOrigin`：

```javascript
const OriginalImage = window.Image;
window.Image = function(...args) {
  const img = new OriginalImage(...args);
  img.crossOrigin = "anonymous";
  return img;
};
```

### 3. 手机上显示太占屏幕

加个判断，屏幕太小就不加载：

```javascript
if (screen.width < 768) return;
```

### 4. 用了 PJAX 之后每次切页都重新加载

把看板娘的代码放到 PJAX 刷新区域外面就好了。

## 想用自己的模型？

默认用的是 [fghrsh 的模型 API](https://github.com/fghrsh/live2d_api)，里面有不少模型可以选。

如果想用自己的模型，可以 fork 那个仓库，把模型放进去，然后改一下 `cdnPath` 指向你自己的地址。

## 参考链接

- [live2d-widget 项目地址](https://github.com/stevenjoezhang/live2d-widget)
- [模型资源 live2d_api](https://github.com/fghrsh/live2d_api)
- [更多模型资源](https://github.com/zenghongtu/live2d-model-assets)

---

好了，就这些。虽然只是个小玩意，但折腾的过程还是挺有意思的。博客瞬间感觉有灵魂了（误

如果遇到问题可以留言，我看到会回复的 👋
