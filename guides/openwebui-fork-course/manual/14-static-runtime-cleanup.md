# 静态描述文件清理：先确认动态路由接管

## 目标

删除已经由后端动态接口接管的旧 `manifest.json` 和 `opensearch.xml`，避免旧品牌描述继续被浏览器读取。

## 对应提交

66f8b06ea

## 为什么这一节重要

- 清理文件不是看见旧文件就删。这个项目里 `image-placeholder.png` 仍被富文本图片组件引用，Leaflet marker 图片也可能被地图组件使用。如果一股脑删除，就会制造新 bug。
- 真正可以清理的是已经有动态路由接管的描述文件：`/manifest.json` 和 `/opensearch.xml`。它们的旧静态版本还写着旧品牌或空对象，留在仓库里只会让未来维护者困惑。
- 这节课的价值在于“保守清理”：先找引用，再找替代入口，最后只删确认过的 tracked 文件。

## 关键文件地图

- `backend/open_webui/main.py`：动态 `/manifest.json` 和 `/opensearch.xml` 路由。
- `src/app.html`：manifest 入口仍指向动态接口。
- `static/manifest.json`：被删除的旧静态描述文件。
- `static/opensearch.xml`：被删除的旧搜索描述文件。

## 手工复现流程

1. 用 `rg` 查所有候选文件名，确认谁仍被源码引用。
2. 发现 `image-placeholder.png` 仍被编辑器引用，就从清理范围移出。
3. 发现 marker 图片可能属于 Leaflet 默认资源，也从清理范围移出。
4. 确认后端已经有 `/manifest.json` 和 `/opensearch.xml` 动态路由。
5. 只 `git rm` 旧静态描述文件，不提交本地临时占位文件。
6. 文档里记录为什么没有删除其他静态资源。

## 常用命令骨架

```bash
git status --short
git add backend/open_webui/main.py
git diff --cached
git diff --cached --check
git commit -m "feat(fay): 静态清理"
git push origin codex/fay-openwebui-custom
```

## 知识课

- 删除文件前要证明“没有引用”或“已有权威替代”。
- 静态描述文件和静态图片资源不是同一种风险。
- 浏览器可能缓存旧 manifest/opensearch，所以动态响应要更可信。
- 清理提交也需要文档，因为它解释的是边界判断。

## 验证门禁

- `backend/open_webui/main.py` 存在两个动态路由。
- `static/image-placeholder.png`、marker 图片、robots 没有被误删。
- `git diff --cached --check` 通过。
- 公开教程不含真实私密路径或凭据。

## 常见坑

- 不要把“旧”当成“无用”。
- 不要把删除资源做成大扫除提交。
- 不要顺手提交 0 字节本地临时文件。
- 不要忘记解释为什么只删两个文件。

## 练习

从一个项目里挑 5 个静态文件，分别写出“仍被引用、已有替代、未知来源、可删除、必须保留”的判断证据。

## 连续阅读

- 上一节：13. 全源码部署：服务器覆盖模板要收口
- 下一节：15. 文档大纲与图层式重排：从笔记目录到通用编辑器模式
