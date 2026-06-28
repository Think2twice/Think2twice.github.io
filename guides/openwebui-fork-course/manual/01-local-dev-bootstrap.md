# 01. 本地开发启动底座

## 目标

把本机 OpenWebUI、dual API gateway、本地配置库和模型分级种子串成一条稳定启动链路。这样以后调试功能时，先在本机闭环验证，再决定是否构建镜像或同步到服务器。

## 这次改动解决什么问题

旧做法容易把本机调试和服务器隧道混在一起：启动 OpenWebUI 时默认去连服务器 gateway，PDF/RAG/TTS 配置可能被数据库里的旧值覆盖，模型列表也要靠手工点后台配置。

新的本地启动底座把这几件事拆清楚：

- 默认启动本机 gateway，不再静默连服务器。
- `.env.fay-local` 用 dotenv 解析器加载，避免 JSON 配置被 shell 解析坏。
- 启动前把 PDF、MinerU、RAG、TTS、STT 配置同步进本地 `webui.db`。
- 启动前应用 GPT-5.5 Auto、Instant、Thinking、Pro 等本地模型分级种子。
- 停止脚本只清理自己管理的 screen、gateway、tunnel 和本地 uvicorn 进程。

## 手工复现流程

1. 准备本地 Python 环境：

   ```bash
   uv venv --python 3.11 .venv
   uv pip install --python .venv/bin/python -r backend/requirements.txt
   ```

2. 准备 `.env.fay-local`，把 OpenWebUI 的 OpenAI-compatible 地址指向本机 gateway：

   ```text
   OPENAI_API_BASE_URL=http://127.0.0.1:13280/v1
   OPENAI_API_BASE_URLS=http://127.0.0.1:13280/v1
   IMAGES_OPENAI_API_BASE_URL=http://127.0.0.1:13280/v1
   IMAGES_EDIT_OPENAI_API_BASE_URL=http://127.0.0.1:13280/v1
   ```

3. 从模板生成本机 gateway 私有配置：

   ```bash
   mkdir -p .local-run
   cp fay-custom/server-overrides/dual-api-gateway/config.example.json .local-run/gateway.config.private.json
   ```

4. 只在 `.local-run/gateway.config.private.json` 和 `.env.fay-local` 中填真实 key。不要把 key 写入 Git。

5. 启动本机闭环：

   ```bash
   fay-custom/scripts/start-local-openwebui.sh --screen
   ```

6. 验证两个本机入口：

   ```bash
   curl -fsS http://127.0.0.1:13280/health
   curl -fsS http://127.0.0.1:8080/health
   ```

7. 停止本机进程：

   ```bash
   fay-custom/scripts/stop-local-openwebui.sh
   ```

## 关键知识

本地调试不能只看 `.env`。OpenWebUI 会把很多后台配置写进 `webui.db` 的 `config` 表。如果数据库里有旧配置，单纯改 `.env.fay-local` 可能没有效果。所以这次增加 `sync-local-rag-config.py`，把关键环境变量同步进配置库。

模型分级也不能只靠前端展示。`apply-local-model-tiers.py` 会写入 `model`、`access_grant`、用户默认设置和新用户触发器，让普通用户打开页面时就能看到稳定的 GPT-5.5 菜单。

如果需要和服务器旧 gateway 做只读对照，才显式使用：

```bash
LOCAL_GATEWAY_MODE=tunnel fay-custom/scripts/start-local-openwebui.sh --screen
```

默认不要用 tunnel 模式，否则本地源码改动和服务器运行态很容易混在一起。

## 验证门禁

- `bash -n fay-custom/scripts/start-local-openwebui.sh fay-custom/scripts/stop-local-openwebui.sh`
- `python -m py_compile fay-custom/scripts/sync-local-rag-config.py fay-custom/scripts/apply-local-model-tiers.py`
- 本机 gateway `/health` 返回成功。
- 本机 OpenWebUI `/health` 返回成功。
- `git diff --cached --check` 通过后再提交。
