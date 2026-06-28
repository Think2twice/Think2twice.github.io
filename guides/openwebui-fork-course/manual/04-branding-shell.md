# ChatGPT 品牌壳：不是只换一个图标

## 目标

把可见品牌从 Open WebUI 收口为 ChatGPT 风格，包括标题、favicon、PWA、manifest、opensearch、通知标题和启动画面。

## 对应提交

69e5f08aa / b09f0d9b0

## 为什么这一节重要

- 品牌迁移最容易做成“换了一个 logo 就算完成”。真实浏览器里，用户会同时看到标签页标题、启动页、PWA 图标、通知标题、搜索描述、缓存里的旧 favicon、移动端主屏幕图标。任何一个残留都会让体验像拼接出来的。
- 本项目里品牌壳还承担一个更深的作用：它让用户知道当前进入的是 Fay 定制的聊天站，而不是官方 OpenWebUI 默认面。这个识别必须稳定。
- 品牌处理也要注意公开边界。课程可以讲结构和方法，不能把真实私有域名、服务器路径和密钥写进公开页面。

## 关键文件地图

- `src/app.html`：HTML title、manifest、favicon 和启动脚本入口。
- `backend/open_webui/config.py` / `env.py`：默认 WEBUI_NAME。
- `backend/open_webui/main.py`：动态 manifest 和 opensearch 响应。
- `fay-custom/server-overrides/open-webui/branding/`：部署品牌补丁和图标资源。

## 手工复现流程

1. 列出所有用户可见品牌面：标题、图标、PWA、通知、搜索、启动页、配置接口。
2. 源码默认值先改，运行态 DB 或启动补丁再负责把旧配置迁过来。
3. favicon 和 manifest 加版本号，降低旧缓存影响。
4. 把 opensearch 从静态旧文件迁到动态路由，避免旧品牌描述残留。
5. 每次重新 build 后跑品牌词扫描，确认可见页面没有回退。

## 常用命令骨架

```bash
git status --short
git add src/app.html
git diff --cached
git diff --cached --check
git commit -m "feat(fay): 品牌壳"
git push origin codex/fay-openwebui-custom
```

## 知识课

- 浏览器品牌面是多入口系统，不是一个文件。
- PWA 图标和 favicon 有独立缓存，很多时候刷新页面不会立刻刷新图标。
- 后端动态 manifest 能把品牌描述和运行配置绑定起来，避免静态文件漂移。
- 启动画面里的旧主题颜色也属于品牌，不只是正文界面。

## 验证门禁

- 首页 `<title>` 是预期品牌。
- `/api/config` 返回的 name 正确。
- `/manifest.json` 和 `/opensearch.xml` 不含旧品牌。
- 页面源代码不再出现明显旧品牌残留。

## 常见坑

- 不要只换 `favicon.ico`，忽略 `apple-touch-icon` 和 web manifest。
- 不要忘记通知标题和启动页。
- 不要把上游多语言包做无意义大面积替换。
- 不要把真实部署域名写进公开教程。

## 练习

打开任意一个 Web App，列出你能看到品牌的十个位置。你会发现品牌迁移从来不是一个文件的事。

## 连续阅读

- 上一节：03. 模型分级与 Arena：可见性不是只改模型表
- 下一节：05. iOS Safari/PWA 单击与输入修复
