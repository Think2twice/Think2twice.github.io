# 网关控制页：状态刷新不能测试上游

## 目标

把临时 gateway 控制入口迁入 OpenWebUI 管理后台，并把“读状态”和“测试接口”彻底分开。

## 对应提交

4f109cef3

## 为什么这一节重要

- 网关控制页看起来只是一个管理页面，实际牵涉权限、服务器内网地址、上游接口、Cloudflare 错误和用户额度风险。一个刷新按钮如果顺手调用真实上游，就可能在用户没意识到时消耗接口额度。
- 正确的产品设计是把状态读取做成低风险、只读、可自动刷新的动作；把接口测试做成明确的管理员手动动作。这样页面能常开，风险动作必须用户有意识地点。
- 这也是后端边界设计课：前端页面不应该直接拼真实上游地址，OpenWebUI 后端负责鉴权和调用 gateway 管理接口，gateway 再只返回脱敏状态。

## 关键文件地图

- `src/routes/(app)/admin/gateway/`：管理员页面入口。
- `backend/open_webui/routers/fay_gateway.py`：OpenWebUI 后端代理和权限检查。
- `fay-custom/server-overrides/dual-api-gateway/`：服务器部署网络示例。
- `fay-custom/LESSONS.md`：记录“刷新不能测试上游”的防复发规则。

## 手工复现流程

1. 先定义页面能做的动作：读取状态、切换模式、手动测试。不要把三者混在一个请求里。
2. 后端新增只读 status 接口，返回当前模式、接口标签、健康摘要，不返回 key、admin token 或完整配置。
3. 前端加载页面时只调用 status；切换和测试按钮单独放在需要管理员确认的区域。
4. 部署时配置容器内网地址，而不是让 OpenWebUI 容器访问自己的 `127.0.0.1`。
5. 验证未登录是 401，普通用户没有管理权限，管理员可以读状态。

## 常用命令骨架

```bash
git status --short
git add src/routes/(app)/admin/gateway/
git diff --cached
git diff --cached --check
git commit -m "feat(fay): 网关控制页"
git push origin codex/fay-openwebui-custom
```

## 知识课

- 管理页的默认动作必须是安全动作。自动刷新、页面进入、tab 切换都不能触发真实上游测试。
- 容器里的 `127.0.0.1` 指向容器自己，不是宿主机，也不是另一个容器；服务互调要用 Docker 网络名或显式内网地址。
- 脱敏状态接口是系统运维里很重要的模式：它让你能观察系统，又不泄露系统。
- 权限不只在前端隐藏按钮，必须在后端接口再校验。

## 验证门禁

- 未登录请求 `/api/v1/fay/gateway/status` 返回 401。
- 管理员页面打开时只读 status，不触发 `/admin/test`。
- 状态摘要不包含 key、token、password、admin 字段。
- 服务器部署中 OpenWebUI 到 gateway 走内网地址。

## 常见坑

- 不要用 Cloudflare 502 这种外部现象直接判断 gateway 坏了，先查容器互调地址。
- 不要在日志里打印完整 gateway 配置。
- 不要让页面刷新等于“帮我测接口”。
- 不要把兼容旧入口当成新的正式入口。

## 练习

把一个危险按钮拆成两个接口：一个只读状态，一个执行动作。写出每个接口允许在什么时候自动调用。

## 连续阅读

- 上一节：01. 本地开发启动底座：先让源码可运行
- 下一节：03. 模型分级与 Arena：可见性不是只改模型表
