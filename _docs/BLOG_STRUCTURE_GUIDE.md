# MyBlog Structure & Configuration Guide (Minimal Mistakes Theme)

> 本文档用于规范 MyBlog 的文件结构、写作规范与主题配置。  
> 适用于 Minimal Mistakes Jekyll 主题，便于后续统一风格与维护。

---

## 📁 目录结构

```
MYBLOG/
│
├── _config.yml              # 全局配置文件
├── Gemfile                  # Ruby 依赖
├── index.html               # 首页布局文件
│
├── _data/
│   ├── navigation.yml       # 顶部导航
│   ├── authors.yml          # 作者信息
│
├── _pages/
│   ├── about.md             # 关于页面
│   ├── categories.md        # 分类索引页
│   ├── tags.md              # 标签索引页
│
├── _posts/
│   ├── YYYY-MM-DD-title.md  # 博客文章（命名规范见下）
│
├── assets/                  # 图片、CSS、JS
└── _site/                   # Jekyll 自动生成的网站（无需手动修改）
```

---

## 🧩 文章命名规范

```
YYYY-MM-DD-英文标题.md
```

例如：
```
2025-11-02-intro-to-kmeans.md
```

> - 文件名必须包含日期；
> - 标题用英文小写短词连接；
> - 中文标题在 Front Matter 中填写。

---

## 🏷️ Front Matter 标准模板

每篇文章头部需包含：

```yaml
---
title: "K-Means 聚类算法原理与实践"
date: 2025-11-02
categories:
  - DataScience
  - DataMining
tags:
  - MachineLearning
  - Clustering
layout: single
author_profile: true
read_time: true
show_date: true
comments: false
related: true
share: true
---
```

### 字段解释：

| 字段 | 类型 | 说明 |
|------|------|------|
| `title` | string | 页面标题 |
| `date` | YYYY-MM-DD | 发布时间 |
| `categories` | list | 分类路径（支持多层） |
| `tags` | list | 标签（非层级） |
| `layout` | string | 页面布局（见下方 Layout 参数） |
| `author_profile` | bool | 是否显示作者信息 |
| `read_time` | bool | 显示阅读时长 |
| `show_date` | bool | 是否显示日期 |
| `comments` | bool | 是否启用评论系统 |
| `related` | bool | 是否显示相关文章 |
| `share` | bool | 是否显示社交分享按钮 |

---

## 🎨 Layout 布局选项

| 布局名 | 说明 |
|---------|------|
| `home` | 博客首页（显示文章列表） |
| `single` | 普通单页（用于文章、About 等） |
| `archive` | 文章归档页 |
| `categories` | 分类页 |
| `tags` | 标签页 |
| `splash` | 带横幅封面的展示页（适合做首页/个人主页） |
| `collection` | 作品集布局（需配合 collection 数据） |

示例（首页 `index.html`）：
```html
---
layout: home
author_profile: true
paginate: 6
---
```

或（个性化主页）：
```html
---
layout: splash
title: "Welcome to MyBlog"
author_profile: true
excerpt: "A space for AI, Data, and Creativity"
---
```

---

## 🧱 Skin 主题皮肤选项

在 `_config.yml` 设置：

```yaml
minimal_mistakes_skin: "mint"
```

可选值：

| 皮肤名 | 描述 |
|---------|------|
| `default` | 默认浅色风格 |
| `air` | 极简清爽 |
| `aqua` | 蓝色调 |
| `contrast` | 对比强烈的灰色调 |
| `dark` | 深色模式 |
| `dirt` | 复古棕色调 |
| `mint` | 青绿色 |
| `neon` | 明亮霓虹风 |
| `plum` | 紫色系 |
| `sunrise` | 温暖橙色系 |

---

## 🗂️ 分类层级规范

推荐两级结构，便于拓展：

| 一级分类 | 二级分类 | 示例 |
|-----------|-----------|------|
| `AI` | `DeepLearning`, `NLP`, `ComputerVision` | `categories: [AI, NLP]` |
| `DataScience` | `DataMining`, `DataAnalysis`, `BigData` | `categories: [DataScience, DataMining]` |

> 💡 一级分类统一使用英文；二级分类精确主题。

---

## 🧭 导航栏配置

`_data/navigation.yml`：

```yaml
main:
  - title: "Home"
    url: /
  - title: "Categories"
    url: /categories/
  - title: "Tags"
    url: /tags/
  - title: "About"
    url: /about/
```

---

## 🏠 首页可选配置

### 1️⃣ 博客文章流式首页
```html
---
layout: home
author_profile: true
paginate: 6
---
```

### 2️⃣ 分类总览页首页
```html
---
layout: categories
title: "文章分类"
permalink: /
author_profile: true
---
```

### 3️⃣ 个性化介绍页首页
```html
---
layout: single
title: "Welcome to MyBlog"
author_profile: true
classes: wide
---
<p>Welcome 👋 I'm Li.</p>
<p>This blog explores AI, data, and creative experiments.</p>
```

---

## 📚 附加页面模板

#### `/pages/about.md`
```markdown
---
title: "About Me"
permalink: /about/
layout: single
author_profile: true
---

Hi 👋 I'm Li.  
I write about **AI**, **Data Science**, and creative coding.
```

#### `/pages/categories.md`
```markdown
---
title: "所有分类"
layout: categories
permalink: /categories/
author_profile: true
---
```

#### `/pages/tags.md`
```markdown
---
title: "所有标签"
layout: tags
permalink: /tags/
author_profile: true
---
```

---

##  `_config.yml` 关键参数

```yaml
title: "MyBlog"
email: your_email@example.com
description: "A personal blog about AI, Data, and Development."
url: "https://localhost:4000"
baseurl: ""
theme: minimal-mistakes-jekyll
minimal_mistakes_skin: "mint"

markdown: kramdown
paginate: 6
paginate_path: "/page:num/"
timezone: Europe/Paris

plugins:
  - jekyll-feed
  - jekyll-seo-tag
  - jekyll-paginate
  - jekyll-archives

defaults:
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: false
      share: true
      related: true
```

---

##  写作与调试流程

1. 新建 Markdown 文件于 `_posts/`
2. 按 Front Matter 模板填写元数据
3. 本地运行：
   ```bash
   bundle exec jekyll serve
   ```
4. 访问 [http://localhost:4000](http://localhost:4000)
5. 检查分类、标签与首页展示效果
6. 确认无误后再推送至 GitHub Pages

---

## 🌱 建议的后续扩展

-  添加 `_drafts/` 文件夹存放草稿；
-  添加 `projects.md` 展示项目或作品；
-  启用评论（Disqus / Utterances）；
-  增加多语言结构 `_i18n/`；
-  自定义首页横幅（使用 `splash` 布局）；
-  添加 `rss` feed 与 sitemap。

---

> **作者建议：**  
> 所有分类、标签、文件命名、Front Matter 均应保持英文，正文可使用中英文混合。  
> 推荐先在本地测试主题与排版，确认后再部署到 GitHub Pages。

常用 type（可选 scope）
feat：新功能
fix：修复 bug
docs：文档
style：格式、空格、分号（不影响代码逻辑）
refactor：代码重构（非新增、非修复）
perf：性能优化
test：测试相关
chore：杂项（构建、脚本、工具）
ci：持续集成相关
build：构建系统或依赖相关
---

*Created by Li — Minimal Mistakes Blog Standard v1.0*
