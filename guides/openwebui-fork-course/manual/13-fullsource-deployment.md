# 13. 全源码部署要把服务器覆盖模板收口

## 目标

把服务器上的 OpenWebUI 覆盖配置、品牌补丁、模型分级、RAG/TTS/STT 环境变量和 Nginx 入口整理成脱敏模板，保证本地源码和服务器部署不再是两套口径。

## 这次改动解决什么问题

如果只在服务器容器里手工 patch，后面会出现三个风险：

- 重新构建或换机器后，不知道哪些文件要挂载。
- 品牌、模型分级、RAG、音频、网关配置散落在服务器上。
- 公开仓库容易误写真实密钥或真实域名。

新的部署模板把这些内容放到 `fay-custom/server-overrides/`：

- `open-webui/fullsource-runtime.Dockerfile`：基于已有镜像复制本地 build/backend。
- `open-webui/docker-compose*.example`：脱敏 compose 模板。
- `open-webui/branding/`：启动前品牌和模型分级 patch。
- `open-webui/patches/live/`：仍需服务器挂载的运行时补丁。
- `nginx/openwebui.conf.example`：脱敏 Nginx 反代示例。
- `dual-api-gateway/docker-compose.override.yml`：网关容器网络示例。

## 手工复现流程

1. 先构建本地源码产物。

2. 用 `fullsource-runtime.Dockerfile` 把 `build/` 和 `backend/` 复制进运行镜像。

3. 启动前运行品牌补丁：

   ```bash
   python /branding/brand_patch.py && bash start.sh
   ```

4. 在 compose 里只写脱敏变量名，真实值放服务器 `.env`：

   ```text
   MINERU_API_KEY
   RAG_OPENAI_API_KEY
   AUDIO_TTS_OPENAI_API_KEY
   AUDIO_STT_OPENAI_API_KEY
   ```

5. Nginx 只暴露 HTTPS 域名入口，`/__fay/*` 映射到补丁目录。

6. 网关管理页走 Docker 内网地址，不用公开域名刷新上游接口。

## 验证门禁

- `python3 -m py_compile backend/open_webui/config.py fay-custom/server-overrides/open-webui/branding/brand_patch.py`
- `python3 -m py_compile` live patch 里的 `builtin.py` 和 `middleware.py`
- 模板里不能出现真实 token、真实私有域名、真实账号密码。
- Nginx 示例里 `__fay` 补丁资源有独立 cache header。
- compose 示例里密钥只以 `${NAME:-}` 形式出现。

## 不要做什么

- 不要把服务器真实 `.env`、数据库、Cookie 或 token 写进仓库。
- 不要把裸 IP:端口当最终入口。
- 不要让刷新网关状态时顺手测试真实上游 API。
