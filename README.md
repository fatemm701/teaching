# 数学讲义静态站

Obsidian（Webpage Export 插件）导出的静态站点，已适配为**站点根托管**，可直接放到任意域名的根目录。

- `index.html` —— 首页（原 `首页.html` 改名）
- `讲义/` —— 讲义正文
- `figs/` —— 矢量图与图的嵌入页
- `site-lib/` —— 样式、脚本、字体、metadata

## 当前部署

- 线上地址：<https://teaching.gyf-time.cn/>
- 服务器：`62.234.12.76`，站点目录 `/home/ubuntu/teaching`
- 容器：`teaching-math`（`nginx:alpine`，只读挂载站点目录，监听 `8090`）
- 反向代理：Nginx Proxy Manager 中 `teaching.gyf-time.cn` → `172.17.0.1:8090`

## 重新发布

在本工程的上一级目录（`E:\Files\019-Part\Math`）执行：

```powershell
powershell -File tools\deploy.ps1
```

脚本会：打包 `output/`（排除 `.git`）→ scp 到服务器 `/tmp` → 解压到 `teaching.new` →
原子替换 `/home/ubuntu/teaching` → `docker restart teaching-math` → 跑一遍 `tools/verify.sh` 校验。

> 注意：**替换目录后必须重启容器**，否则 bind mount 仍指向旧 inode，页面会全部 404。

## 对导出结果的适配

1. `<head>` 里的资源地址改成真相对路径（子目录页面自动加 `../`），不再依赖 `<base href=".">`。
2. `site-lib/metadata.json`、`search-index.json` 中的站点域名改为相对路径。
3. `site-lib/scripts/webpage.js`：
   - 新增 `getRootURL()` / `getLibURL()`，用 `document.currentScript.src` 反推站点根；
   - `loadWebsiteData()` 改用 `getLibURL()` 读取 `metadata.json`；
   - `fetch()` 统一走站点根绝对路径，不再受当前页面目录或 `<base>` 影响。
4. `site-lib/scripts/graph-wasm.js`：wasm 路径同样基于 `document.currentScript.src` 定位。
5. 所有页面的 `<meta name="pathname">` 统一为站点根相对形式，保证入口页能在 `metadata.webpages` 中命中。
6. 正文 `internal-link` 的 `href` 统一为站点根相对形式（运行时用它精确匹配 metadata 键）。
7. 补上缺失的空片段 `site-lib/html/custom-head-content-content.html`。

### 代理层注意

Nginx Proxy Manager 的「Assets」缓存段（`conf.d/include/assets.conf`）在本服务器上会让所有
`css/js/svg/ttf/…` 请求返回 502，已在 `proxy_host/13.conf` 中注释掉该 `include`。
重新生成该 proxy host 时这个 `include` 会回来，需要再次注释。

### 旧版本

旧的纯 Node 版站点（`teaching-math:latest` 镜像，内容是 `subjects/math/**/lecture.html` 那一套）
已停用，源码仍在服务器 `/home/ubuntu/apps/teaching`。