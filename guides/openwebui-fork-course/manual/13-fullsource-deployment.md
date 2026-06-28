# 全源码部署：服务器覆盖模板要收口

## 目标

把服务器上的 full-source 镜像、compose、Nginx、品牌补丁、RAG/TTS/STT 配置和 gateway 网络示例脱敏回收到源码仓库。

## 对应提交

513e0aa9f

## 为什么这一节重要

- 服务器上手工 patch 最危险的地方，是它会让项目出现两套真相：本地源码一套，服务器运行态一套。只要重建镜像、换机器或升级上游，就不知道哪些改动该带过去。
- 全源码部署的目标是让服务器只是运行源码生成的交付物。真实密钥和数据卷留在服务器，结构、模板、补丁和验证规则回到仓库。
- 这节课也是部署边界课：compose project 名、external volume、Nginx 缓存头、动态 API no-store、静态资源 immutable、gateway 内网地址，都要明确。

## 关键文件地图

- `fay-custom/server-overrides/open-webui/fullsource-runtime.Dockerfile`：复制 build/backend 的运行镜像。
- `fay-custom/server-overrides/open-webui/docker-compose*.example`：脱敏 compose 模板。
- `fay-custom/server-overrides/nginx/openwebui.conf.example`：Nginx 反代和缓存示例。
- `fay-custom/server-overrides/open-webui/branding/brand_patch.py`：启动前品牌和模型补丁。

## 手工复现流程

1. 先在本地构建前端和后端产物，确认 build 版本。
2. 用 fullsource runtime Dockerfile 把源码产物复制进运行镜像。
3. compose 模板只写变量名和占位符，真实 key 留在服务器 `.env`。
4. 固定 external volume 名，避免 compose project 名变化创建空数据卷。
5. Nginx 对 `/_app/immutable/` 做长缓存，对 `/api/` 做 no-store。
6. 部署后做三向核对：本地源码、服务器 source-build、容器运行文件。

## 常用命令骨架

```bash
git status --short
git add fay-custom/server-overrides/open-webui/fullsource-runtime.Dockerfile
git diff --cached
git diff --cached --check
git commit -m "feat(fay): 全源码部署"
git push origin codex/fay-openwebui-custom
```

## 知识课

- Docker Compose project 名会影响卷和网络名，不确认就重建可能像“数据丢了”。
- 全源码镜像应该包含完整 build/backend，不靠服务器 bind mount 拼半套。
- 静态资源可以长缓存，登录态页面和 API 不能全站缓存。
- 服务器补丁只有反补到源码模板里，才算长期治理完成。

## 验证门禁

- 容器 healthy，公网 `/health` 正常。
- 公网 `/_app/version.json` 和容器 build version 一致。
- Nginx 配置语法通过，缓存头符合动态/静态边界。
- 模板扫描不含真实 token、域名、IP、cookie 或密码。

## 常见坑

- 不要在服务器热改后忘记反补源码。
- 不要让 compose 自动创建新空卷。
- 不要把裸 IP:端口当正式交付入口。
- 不要为了加速缓存登录态 HTML 或 API。

## 练习

写一份部署检查表，至少包含镜像、容器、数据卷、Nginx、缓存头、API config、前端版本七项。

## 连续阅读

- 上一节：12. Workspace seeds：资源包要能重复安装和回滚
- 下一节：14. 静态描述文件清理：先确认动态路由接管
