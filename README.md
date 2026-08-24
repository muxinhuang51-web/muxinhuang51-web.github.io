# 山止川行

黄慕鑫的个人主页，使用 Astro 构建，计划部署到 GitHub Pages。

## 本地开发

```powershell
npm install
npm run dev
```

## 更新内容

- 个人信息：编辑 `src/config/site.ts`
- 新增札记：在 `src/content/posts/` 新建 Markdown 文件
- 新增研究方向或项目：在 `src/content/projects/` 新建 Markdown 文件

提交并推送后，GitHub Pages 会在部署配置完成后自动更新。

## 发布到 GitHub Pages

1. 在 GitHub 新建空仓库 `muxinhuang51-web.github.io`
2. 不要勾选自动生成 README、`.gitignore` 或许可证
3. 在本地添加远程仓库并推送 `main` 分支
4. 在仓库的 **Settings → Pages** 中确认 Source 为 **GitHub Actions**

站点将发布到 <https://muxinhuang51-web.github.io>。
