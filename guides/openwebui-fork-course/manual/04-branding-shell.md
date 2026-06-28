# 04. ChatGPT 品牌壳与 PWA 图标

## 目标

把 OpenWebUI 的可见外壳改成 Fay 自用的 ChatGPT 风格：浏览器标签、API 配置名、通知标题、favicon、PWA manifest、启动页 splash 和静态图标都一致。

## 这次改动解决什么问题

只改页面上的一个标题不够。OpenWebUI 的品牌会从很多入口露出来：

- 后端 `/api/config` 返回的 `name`。
- FastAPI 标题和启动日志。
- `src/app.html` 里的浏览器 title、favicon、apple touch icon、manifest。
- PWA 安装后的图标和 splash。
- 浏览器通知标题。
- 静态目录里的图标副本和服务器覆盖目录。
- 上游 `her` 主题可能把启动页变成红色。

所以品牌提交必须是“可见面全覆盖”，不是单点替换。

## 手工复现流程

1. 修改后端默认名称：

   ```text
   backend/open_webui/env.py
   backend/open_webui/main.py
   ```

2. 修改前端 HTML 壳：

   ```text
   src/app.html
   ```

   重点是 `<title>`、favicon、apple touch icon、manifest 版本号和 splash 图片版本号。

3. 修改通知标题：

   ```text
   src/routes/+layout.svelte
   ```

   不要写死 `Open WebUI`，而是使用当前 `$WEBUI_NAME`。

4. 同步静态图标资源：

   ```text
   backend/open_webui/static/
   static/
   static/static/
   ```

5. 清理前端常量和可见文案：

   ```text
   src/lib/constants.ts
   src/lib/components/**/*
   ```

   `APP_NAME`、设置页、分享页、社区提示、页面标题里的旧品牌都要扫一遍；但上游仓库链接、许可证和兼容字段不要粗暴替换。

6. 如果保留服务器启动补丁，品牌资源也要同步到：

   ```text
   fay-custom/server-overrides/open-webui/branding/
   ```

7. 去掉 `her` 启动页分支或把它重置为系统主题，避免旧缓存让 Fay 看到红色 splash。

## 验证门禁

- `/api/config` 返回 `name=ChatGPT`。
- 首页 HTML 有 `<title>ChatGPT</title>`。
- 首页 HTML 指向 ChatGPT 图标资源并带 cache-bust query。
- manifest 或 webmanifest 的 `name` 和 `short_name` 都是 `ChatGPT`。
- 浏览器通知标题使用 `$WEBUI_NAME`。
- 前端 `APP_NAME` 和设置/分享/社区相关可见文案不再显示旧品牌。
- 搜索源码中仍出现 `Open WebUI` 的地方时，要判断是上游说明文字、外部链接、兼容字段，还是漏改的可见面。

## 不要做什么

- 不要把官方 OpenWebUI 仓库链接、许可证、上游文档说明全部粗暴改掉。
- 不要提交真实账号、域名、key 或部署私有信息到公开教程。
- 不要只改 `src/app.html` 后就说完成。
- 不要让旧 `her` 主题继续控制启动页。
