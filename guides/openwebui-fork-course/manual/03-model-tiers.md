# 模型分级与 Arena：可见性不是只改模型表

## 目标

让普通用户也能看到 GPT-5.5 五档模型和 Arena Model，并把模型顺序、默认 pin、权限和部署模板统一。

## 对应提交

01125a2eb

## 为什么这一节重要

- 模型下拉框不是一个静态列表。OpenWebUI 同时有 model 表、配置型 evaluation arena models、用户 settings、默认 pinned models、排序列表和权限 grant。只改其中一层，就会出现管理员能看、普通用户看不到，或者刷新后顺序又乱了。
- 本项目要做的是一个接近 ChatGPT 的产品壳：用户不应该看见一堆上游内部模型名，而应该看到稳定、可解释的档位，比如 Instant、Thinking、Pro 等。
- Arena Model 的坑在于它不一定存在于普通 model 表里，而是配置型模型；普通用户可见性要看 `meta.access_grants`。这类问题如果只看 UI，会误以为前端下拉坏了。

## 关键文件地图

- `backend/open_webui/config.py`：默认 Arena model 和 access grants。
- `fay-custom/scripts/apply-local-model-tiers.py`：本地模型分级写入脚本。
- `fay-custom/server-overrides/open-webui/branding/brand_patch.py`：服务器启动前修正模型和品牌。
- `fay-custom/server-overrides/open-webui/docker-compose.yml.example`：部署默认开关。

## 手工复现流程

1. 先列出用户应该看到的最终模型顺序，而不是直接翻 DB。
2. 把五档模型写成稳定 id 和显示名，底层 `base_model_id` 再指向真实上游模型。
3. 把 `DEFAULT_PINNED_MODELS`、`MODEL_ORDER_LIST`、Arena 开关和 DB 配置同步。
4. 给 Arena 配置型模型补 `user:*:read`，让普通用户能看到。
5. 用普通 user token 调 `/api/models`，不要只用 admin 页面目测。

## 常用命令骨架

```bash
git status --short
git add backend/open_webui/config.py
git diff --cached
git diff --cached --check
git commit -m "feat(fay): 模型分级"
git push origin codex/fay-openwebui-custom
```

## 知识课

- 显示模型和上游模型是两层：前者是产品语言，后者是供应商路由。
- access grant 是“谁能看到/使用”的权限层，不能用 UI 隐藏代替。
- 启动补丁适合处理运行态配置漂移，但必须反补到源码和部署模板里。
- 模型默认值属于产品体验的一部分，不只是管理员偏好。

## 验证门禁

- 普通用户 `/api/models` 包含五档模型和 `arena-model`。
- compose 模板中 Arena 开关为预期值。
- 模型排序列表和 pinned models 含 `arena-model`。
- 启动补丁和本地脚本生成的配置一致。

## 常见坑

- 不要只改 model 表，忘记配置型 Arena。
- 不要只用管理员账号验收模型可见性。
- 不要把 display name 和真实 upstream id 混为一谈。
- 不要让服务器启动补丁成为唯一真相。

## 练习

设计一个“学生版/老师版/专家版”三档模型下拉，写清每档 display name、base model、权限和默认排序。

## 连续阅读

- 上一节：02. 网关控制页：状态刷新不能测试上游
- 下一节：04. ChatGPT 品牌壳：不是只换一个图标
