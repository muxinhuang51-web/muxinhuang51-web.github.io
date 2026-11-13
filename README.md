# 山止川行

这是一个基于 Astro 的个人博客。文章、动态和简历分别管理，先保持简单，后续可以继续增加项目、标签或其他页面。

## 本地运行

```powershell
npm install
npm run dev
```

Windows PowerShell 如果提示脚本执行策略错误，可以使用：

```powershell
npm.cmd run dev
```

提交到 GitHub 的 `main` 分支后，GitHub Actions 会自动构建并部署。

## 想改什么，去哪里改

### 写博客文章

在 `src/content/posts/` 新建 `.md` 文件。文件名会成为文章地址，例如 `my-post.md` 对应 `/notes/my-post/`。

```md
---
title: 文章标题
description: 一句话介绍（可选）
published: 2026-09-01
tags:
  - 标签
draft: false
---

正文写在这里，支持标准 Markdown。
```

`draft: true` 的文章不会显示在网站中。文章列表会自动按 `published` 从新到旧排列。

### 更新动态轨迹

在 `src/content/updates/` 新建 `.md` 文件。它会自动出现在首页的“我的动态轨迹”时间线中。

```md
---
title: 完成了什么
date: 2026-09-01
type: 学习
description: 用一句话记录这次更新。
link: /notes/my-post/
---
```

`link` 是可选的，可以链接到文章、简历或其他页面。动态也支持 `draft: true` 暂时隐藏。

只确定到月份的经历可以设置 `datePrecision: month`，并将 `date` 写为该月第一天（例如 `2025-11-01`）用于排序；页面只显示年月。

### 修改站点名称、作者和 GitHub

编辑 `src/config/site.ts`：

- `title`：站点名称
- `owner`：作者名称
- `description`：首页简介和默认 SEO 描述
- `github`：导航栏里的 GitHub 地址

### 修改首页结构

编辑 `src/pages/index.astro`。文章列表和动态时间线都在这里渲染。

### 修改文章详情页

编辑 `src/pages/notes/[id].astro` 和 `src/layouts/ArticleLayout.astro`。

### 修改简历

将新的 PDF 覆盖到 `public/resume/resume.pdf`。简历页面代码在 `src/pages/resume/index.astro`，访问地址是 `/resume/`。

### 修改样式

编辑 `src/styles/global.css`。这里包含颜色、间距、字体、文章列表、时间线和移动端适配。

### 添加新的内容类型

在 `src/content.config.ts` 中增加一个 collection，再在 `src/content/` 下建立对应目录。首页或新页面通过 `getCollection("集合名")` 读取它。

## 目录速查

- `src/content/posts/`：博客文章
- `src/content/updates/`：动态轨迹
- `src/content/projects/`：预留的项目内容，暂未展示
- `src/content.config.ts`：所有 Markdown 字段定义
- `src/pages/index.astro`：首页
- `src/pages/notes/[id].astro`：文章详情页
- `src/pages/resume/index.astro`：简历页
- `src/config/site.ts`：站点基本信息
- `src/styles/global.css`：全局样式
- `public/`：不会经过处理的静态文件，例如 PDF 和图片

## 发布前检查

```powershell
npm run check
npm run build
```

两个命令都通过后再提交代码。线上地址是 <https://muxinhuang51-web.github.io/>。
