# 山止川行

一个基于 Astro 的极简个人博客。当前只保留写博客需要的部分：Markdown 文章、文章列表和文章详情页。

## 本地开发

```powershell
npm install
npm run dev
```

打开终端提示的本地地址即可预览。

## 写一篇文章

在 `src/content/posts/` 新建一个 `.md` 文件：

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

`draft: true` 的文章不会出现在网站中。文件名会成为文章地址，例如 `my-post.md` 对应 `/notes/my-post/`。

## 项目结构

- `src/content/posts/`：博客文章
- `src/content.config.ts`：文章字段定义，后续可扩展标签、系列等字段
- `src/pages/index.astro`：文章列表页
- `src/pages/notes/[id].astro`：文章详情页
- `src/config/site.ts`：站点标题、作者和 GitHub 地址

项目内容集合已保留在配置中，等博客基础稳定后可以再增加项目、关于等页面。
