# 数学讲义静态站

Obsidian 导出后做了少量适配，使其可以托管在任意子路径（如 GitHub Pages 项目页 `/<repo>/`）。

- `index.html` —— 首页（原 `首页.html` 改名）
- `讲义/` —— 讲义正文
- `figs/` —— 矢量图与图的嵌入页
- `site-lib/` —— 样式、脚本、字体、metadata

## 部署

本仓库根目录**就是**网站根目录，没有构建步骤，纯静态文件。

在 **Settings → Pages → Source** 选择 **Deploy from a branch**，
分支选 `gh-pages`，目录选 `/ (root)`，保存即可。

站点地址：https://fatemm701.github.io/teaching/

`main` 与 `gh-pages` 两个分支内容一致；改完 `main` 后执行
`git push origin main:gh-pages` 即可重新发布。

## 对导出结果的适配

1. `<head>` 里的资源地址改成真相对路径（子目录页面自动加 `../`），不再依赖 `<base href=".">`。
2. `site-lib/metadata.json` 里以站点域名开头的链接改为相对路径。
3. `site-lib/scripts/webpage.js` 中 `fetch("site-lib/…")` 的解析改为基于页面所在目录，
   修复子目录页面读取 metadata / 搜索索引 404 的问题。
4. 补上缺失的空片段 `site-lib/html/custom-head-content-content.html`。