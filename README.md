# 🚀 Wang Yubo (王昱博) - 个人品牌数字枢纽 (VibeUI x Twilight Edition)

这是一个基于 **AstroPaper** 框架，融合了 **Twilight** 的三栏华丽布局与 **VibeUI (Claude 风格)** 温暖人文设计美学的深度定制博客系统。它不仅是一个内容发布平台，更是你作为南京大学环境科学研究生在 AI × 环境领域学术研究、项目成果和学习笔记的**高阶展示枢纽**。

---

## 🎨 设计哲学与视觉规范

本项目遵循 **`DESIGN.md`** 中定义的视觉准则：
- **人文气息**：采用皮纸色背景 (`#f5f4ed`) 与衬线体标题，营造如文学杂志般的阅读感。
- **三栏式布局**：参考 Twilight 主题，将信息分为 **“身份与导航”**、**“核心内容”** 与 **“动态数据”** 三个维度。
- **微光感 (Glassmorphism)**：所有容器均采用半透明毛玻璃背景、微弱边框 (`ring-1`) 与柔和投影，确保 UI 简约而极具质感。
- **胶囊化信息层级**：日期、标签均采用 Pills 设计，强化信息的视觉区分度。

---

## 🧭 代码结构与网站 UI 深度映射图

为了方便后续维护，以下是文件目录与网页实际模块的精准对应关系：

### 1. 首页 (Homepage) - `src/pages/index.astro`
首页现在被重构为标准的 **三栏式 Grid 布局**：
- **左侧栏 (Left Sidebar)**:
    - `Profile Card`: 展示头像、姓名及社交链接。
    - `Directory`: 自动计数的导航菜单（关联 `sortedPosts.length`）。
    - `Tag Cloud`: 展示最热门的 10 个分类标签（由 `src/utils/getUniqueTags.ts` 生成）。
- **中间主内容区 (Center Content)**:
    - `Hero Area`: 采用渐变文字的身份介绍。
    - `Career Highlights`: 模块化卡片（Education, Projects, Research）。
    - `Post Lists`: 推荐与最新文章列表（调用 `src/components/Card.astro`）。
- **右侧栏 (Right Sidebar)**:
    - `Announcement`: 顶部的动态公告模块。
    - `Activity Heatmap`: **[NEW]** 位于 `src/components/Heatmap.astro`，自动扫描本地 Markdown 并在侧边栏绘制 52 周发布热图。
    - `Statistics`: 全站数据统计面板。

### 2. 列表页 (List Pages) - Timeline Style
- **`src/pages/posts/` & `src/pages/notes.astro`**:
    - 采用了 **Twilight 轴线布局**。在 `<ul>` 容器上增加了 `border-l` 装饰线。
    - 列表项左侧通过绝对定位生成的 `Timeline Dot`（圆点）营造时间轴美感。

### 3. 组件库 (Shared Components)
- **`src/components/Header.astro`**: 重构为 `sticky` 悬浮毛玻璃导航，包含搜索和主题切换。
- **`src/components/Tag.astro`**: 已升级为 VibeUI 胶囊样式，支持 `sm` 尺寸。
- **`src/components/Datetime.astro`**: 升级为带日历图标的 Pills 样式。
- **`src/components/Card.astro`**: 核心内容卡片，增加了 Hover 时的 `translate-y` 位移效果。

---

## 🛠️ 环境搭建与开发流程

### 1. 基础设置
- 修改 **`src/config.ts`**：设置全站 `SITE` 名称、SEO 描述、每页数量。
- 修改 **`src/constants.ts`**：配置你的 GitHub, ResearchGate, X 等社交链接。
- 修改 **`src/styles/global.css`**：在 `:root` 中调整 `--accent` 颜色（当前为学术蓝/陶土红）。

### 2. 本地开发
```bash
# 安装依赖
npm install

# 启动开发服务器（支持 HMR 热更新）
npm run dev
```
访问 `http://localhost:4321` 即可实时预览。

### 3. 部署发布
项目已配置 GitHub Pages 自动构建：
```bash
git add .
git commit -m "feat: new content or style update"
git push
```

---

## 🖋️ 内容创作指南 (Obsidian 深度集成)

`src/data/blog/` 目录即为你的内容库，建议直接将其作为 Obsidian 的一个子文件夹。

### YAML 标准配置
所有 Markdown 文件开头必须包含以下元数据：
```yaml
---
title: "文章标题"
pubDatetime: 2024-05-05T12:00:00Z
tags:
  - notes      # ⭐️ 必填：包含此标签的文章会自动分类到 /notes 导航栏
  - research   # 可选：其他分类
description: "这篇文章的简短摘要"
featured: false # 是否在首页 Featured 栏目展示
draft: false   # 为 true 时不会在线上发布
---
```

---

## 🚀 后续开发路线图 (Future Roadmap)

为了打造一个顶级的求职/学术作品集，你可以从以下方向继续迭代：

### 1. 动态个人名片展示 (Profile Card Pro)
- 在左侧栏 Profile Card 下方增加一个 **“Status”** 小组件，通过脚本自动获取最新的 GitHub Activity 或目前的研究状态。

### 2. 简历与学术成果 (CV & Publications)
- **独立成果页**: 考虑建立 `src/pages/publications.astro`，使用 BibTeX 风格列出你的论文。
- **PDF 预览**: 在 `/about` 页面通过 `<iframe>` 或专用组件直接嵌入并预览你的 PDF 简历。

### 3. 互动增强
- **Giscus 评论区**: 在文章底部开启基于 GitHub Issues 的评论系统，增强学术讨论氛围。
- **搜索优化**: 目前已集成 PageFind，可以进一步美化搜索浮层的毛玻璃样式。

### 4. 自动化与工具链
- **AI 摘要生成**: 在 Obsidian 中使用插件自动为文章生成描述（Description），提升 SEO 效率。
- **微信公众号/知乎同步**: 利用静态资源导出功能，实现多平台内容同步。

---

> “所有的代码都是为了更好的表达。” 祝你的科研与职场生涯如这个博客一般，简洁而华丽，沉稳而有力。 🎓🚀
