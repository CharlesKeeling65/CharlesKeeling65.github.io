# Wang Yubo (王昱博) - 个人博客与求职作品集

这是一个基于 **AstroPaper** 深度定制的个人博客与学术/求职主页。本项目不仅是一个静态博客，更是你展示学术科研成果、项目经历和日常学习笔记的核心数字枢纽。

---

## 🧭 代码结构与网站UI对应指南

为了方便你后续进行个性化修改和开发，以下是当前代码库核心文件与网站页面实际效果的一一对应关系：

### 1. 全局配置 (Global Settings)
- **`src/config.ts`** ➡️ **全站基础设置**
  - 控制网站的标题 (Title)、作者 (Author)、SEO 描述。
  - 设置每页显示的文章数量 (`postPerPage`) 和明暗主题支持 (`lightAndDarkMode`)。
- **`src/constants.ts`** ➡️ **社交链接与分享 (Social Links)**
  - 控制首页和页脚显示的社交媒体图标（如 GitHub, X, ResearchGate 等）。

### 2. 核心页面 (Pages & UI)
- **`src/pages/index.astro`** ➡️ **网站首页 (Homepage)**
  - 包含你的英文自我介绍（Hero 区域）。
  - **Career Highlights** 模块：展示 Education (教育)、Projects (项目)、Research (科研) 的模块化卡片。
  - Featured / Recent Posts 列表。
- **`src/pages/about.md`** ➡️ **关于页面 (/about)**
  - 你详细的中英文自我介绍、教育背景和研究兴趣列表。
- **`src/components/Header.astro`** ➡️ **顶部导航栏 (Navbar)**
  - 定义了全站顶部的导航菜单项（Posts, Notes, Tags, About 等）。
- **`src/pages/notes.astro`** ➡️ **学习笔记聚合页 (/notes)**
  - 自动抓取并展示所有包含 `notes` 标签（tag）的文章。

### 3. 内容与文章 (Content)
- **`src/data/blog/`** ➡️ **Markdown 存放目录**
  - 这是所有文章、笔记存放的核心目录。你可以将其作为本地 Obsidian 库的目录，直接进行无缝写作。

---

## 📝 写作与发布指南 (结合 Obsidian)

本博客采用 Markdown 驱动，非常适合与 Obsidian 联动。

### 撰写新文章或笔记
在 `src/data/blog/` 下创建 Markdown 文件。文件顶部必须包含 **YAML Frontmatter**：

```yaml
---
title: "你的文章/笔记标题"
pubDatetime: 2024-05-04T15:00:00Z  # 发布时间
tags:
  - notes      # ⭐️ 只要包含 notes 标签，就会自动进入顶部导航的 Notes 专栏
  - ai         # 其他自定义分类标签
description: "这篇文章的简短描述，会展示在列表页和用于 SEO。"
featured: false # 如果设置为 true，将会在首页推荐展示
draft: false   # 如果设置为 true，则仅在本地可见，不会线上发布
---
```

### 本地预览与线上发布
1. **本地预览**: 打开终端运行 `npm run dev`，在浏览器访问 `http://localhost:4321` 实时预览你的 Obsidian 笔记效果。
2. **线上发布**: 运行 `git add .` -> `git commit -m "update"` -> `git push`，GitHub Actions 会自动构建并发布你的最新内容。

---

## 🚀 后续可深化的设计与开发空间

随着你的内容不断积累，这个博客还有很大的扩展空间，以下是一些建议的开发方向：

### 1. 完善求职/申请展示 (Portfolio Enhancement)
- **增加独立的 `/projects` 或 `/research` 页面**：
  - 如果项目和论文增多，可以在首页导航栏新增一个 `Projects` 标签。
  - 创建一个瀑布流或卡片阵列的页面，详细介绍每个项目（例如 `Geoprocess` 和 `stateful_interview_agent`）的代码仓库链接、论文 PDF 下载链接和架构图。
- **添加简历下载按钮**：
  - 在 `index.astro` 的 Hero 区域或 `/about` 页面，增加一个醒目的 **"Download CV"** 按钮，链接到存放于 `public/` 目录的 PDF 简历。

### 2. 交互与读者互动
- **引入评论系统 (Giscus / Utterances)**：
  - AstroPaper 天然支持 Giscus。你可以通过配置 GitHub App，在博客底部引入评论区，让同行或访客可以与你交流探讨。*(参考 `src/data/blog/how-to-integrate-giscus-comments.md`)*
- **全站搜索优化**：
  - 目前内置了 PageFind 搜索，你可以进一步定制搜索结果的卡片 UI，使其支持按类别（文章/笔记）快速过滤。

### 3. 自动化与基础设施建设
- **Obsidian 自动化部署**：
  - 在 Obsidian 内安装 **Obsidian Git** 插件，配置快捷键一键 Commit 和 Push，实现完全不离开 Obsidian 就能更新博客的丝滑体验。
- **文章封面自动生成 (Dynamic OG Images)**：
  - 当前主题支持根据标题动态生成社交分享封面。你可以修改 `src/pages/og.png.ts`，定制一张具有你个人品牌色彩和南京大学元素的默认 OG 图片模板。

### 4. 视觉与品牌个性化
- **定制主题色调**：
  - 在 `src/styles/base.css` 中修改 `--color-accent`，可以将主题的高亮色改为你偏好的学术蓝或护眼绿。
- **添加动态徽章**：
  - 在首页的 About 区域，使用 GitHub Readme Stats 引入你的 GitHub 代码提交热图，展示你的开源活跃度。

---
*Happy coding and writing! 祝你在科研与求职中一切顺利！*
