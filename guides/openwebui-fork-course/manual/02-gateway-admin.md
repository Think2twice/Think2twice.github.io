# 02. 网关控制页与上游路由

## 目标

把原本散落在独立 gateway 页面、服务器配置和临时热修里的接口切换能力迁回 OpenWebUI 源码。管理员在 `/admin/gateway` 里能查看当前状态、切换模式、手动测试接口；普通刷新不能偷偷调用上游模型。

## 这次改动解决什么问题

旧做法有三个风险：

- gateway 控制入口在 OpenWebUI 之外，权限和登录态容易割裂。
- 管理页刷新如果顺手测试接口，会在 Fay 没有明确要求时消耗接口额度。
- 图片生成、embedding、普通聊天都走同一套 active 切换，会让图片或 RAG 的一次请求把普通聊天默认接口改掉。

新的设计把控制流分成三层：

- OpenWebUI 后端路由：`/api/v1/fay/gateway/*`，只允许管理员访问。
- OpenWebUI 前端页面：`/admin/gateway`，展示状态和按钮。
- dual API gateway：提供 `/admin/status`、`/admin/mode`、`/admin/test`，并校验 OpenWebUI 管理员登录态或本地 admin key。

## 手工复现流程

1. 在后端新增 router：

   ```text
   backend/open_webui/routers/fay_gateway.py
   ```

   它负责把 OpenWebUI 管理员请求转发给 gateway 的 admin API。服务器里通过 `FAY_GATEWAY_BASE_URL=http://dual-api-gateway:13280` 走 Docker 内网，本机默认走 `http://127.0.0.1:13280`。

2. 在 `backend/open_webui/main.py` 注册 router：

   ```python
   from open_webui.routers import fay_gateway
   app.include_router(fay_gateway.router, prefix="/api/v1/fay/gateway", tags=["fay-gateway"])
   ```

3. 在前端新增 API client：

   ```text
   src/lib/apis/fayGateway.ts
   ```

   页面只调用 OpenWebUI 自己的 `/api/v1/fay/gateway/*`，不要让浏览器直接拿 gateway admin key。

4. 在 Admin 导航加入口：

   ```text
   src/routes/(app)/admin/+layout.svelte
   ```

5. 新增页面：

   ```text
   src/routes/(app)/admin/gateway/+page.svelte
   ```

   刷新按钮只读状态；测试按钮必须用户手点。

6. 在 gateway 本体里补管理员鉴权和路由策略：

   ```text
   fay-custom/server-overrides/dual-api-gateway/gateway.py
   ```

   关键点是 `require_admin()` 既支持本地 admin key，也支持把 OpenWebUI cookie/token 转发给 `openwebui_auth_url` 校验管理员身份。

## 路由策略

普通聊天沿用 gateway active 策略。图片请求和 embedding 请求是特殊流量：

- 图片请求可优先走 `image_preferred`，默认不回退，避免污染官方 ChatGPT 历史或误用另一条接口。
- embedding 请求可优先走 `embedding_preferred`，但不应该改变普通聊天 active。
- 图片和 embedding 的成功或失败不更新普通聊天的 active 上游。

这样 `/admin/gateway` 看到的“当前接口”仍代表普通聊天，不会被一次图片或 RAG 请求悄悄改掉。

## 验证门禁

- `python -m py_compile backend/open_webui/routers/fay_gateway.py fay-custom/server-overrides/dual-api-gateway/gateway.py`
- Svelte parse 能读取 `src/routes/(app)/admin/gateway/+page.svelte`。
- TypeScript 能转译 `src/lib/apis/fayGateway.ts`。
- 未登录请求 `/api/v1/fay/gateway/status` 应返回 401。
- 管理员刷新 `/admin/gateway` 只读状态，不触发 `/admin/test`。
- 只有点击“测试”按钮才允许发极小上游测试请求。

## 不要做什么

- 不要把真实 gateway key 写进源码或教程。
- 不要把公网域名当作 OpenWebUI 到 gateway 的默认内部地址。
- 不要让页面自动轮询 `/admin/test`。
- 不要把图片/RAG 请求的上游切换结果当成普通聊天 active。
