# 03. GPT-5.5 模型分级与 Arena 可见性

## 目标

把模型下拉从“一个上游模型名”整理成更像产品的模型菜单：`GPT-5.5 Auto`、`GPT-5.5`、`GPT-5.5 Instant`、`GPT-5.5 Thinking`、`GPT-5.5 Pro`，并让普通用户也能看到 `Arena Model`。

## 这次改动解决什么问题

模型菜单不是前端文案。OpenWebUI 的可见模型由多个地方共同决定：

- `model` 表中的模型 preset。
- `access_grant` 是否允许普通用户读取。
- 全局 `config.ui.default_models`、`default_pinned_models`、`model_order_list`。
- 用户自己的 `settings` 和新用户默认设置。
- Arena 模型不一定在 `model` 表里，它还存在于 `config.evaluation.arena.models`。

所以如果只改页面文字，刷新、换账号、重启容器后很容易又回到旧状态。

## 手工复现流程

1. 先确定底层真实模型 ID，例如当前基座是：

   ```text
   gpt-5.5
   ```

2. 为五个展示档位建立 preset：

   ```text
   gpt-5.5-auto
   gpt-5.5
   gpt-5.5-instant
   gpt-5.5-thinking
   gpt-5.5-pro
   ```

   其中 Auto/Instant/Thinking/Pro 可以把 `base_model_id` 指向 `gpt-5.5`。

3. 给每个 preset 写 `access_grant`：

   ```text
   resource_type=model
   resource_id=<model_id>
   principal_type=user
   principal_id=*
   permission=read
   ```

4. 更新全局 UI 默认值：

   ```text
   default_models=gpt-5.5-auto
   default_pinned_models=gpt-5.5-auto,gpt-5.5,gpt-5.5-instant,gpt-5.5-thinking,gpt-5.5-pro,arena-model
   model_order_list=[同上]
   ```

5. 更新已有用户 settings，并给新用户加默认设置触发器。

6. Arena 额外处理：在默认 Arena 配置的 `meta.access_grants` 中加入 `user:*:read`。否则管理员能看到，普通用户可能看不到。

## 本地脚本入口

本地启动底座已经把模型分级脚本接入启动流程：

```bash
fay-custom/scripts/apply-local-model-tiers.py --data-dir .local-data/open-webui
```

这个脚本会 upsert 五档模型、写公开读取权限、清理旧模型 ID、更新用户默认设置，并安装新用户默认设置触发器。

## 验证门禁

- 普通用户请求 `/api/models` 能看到五档模型和 `arena-model`。
- `gpt-5.5-auto` 是默认模型。
- Auto/Instant/Thinking/Pro 的 `base_model_id` 指向 `gpt-5.5`。
- 旧 ID `fay-chatgpt2api-gpt-5`、`fay-sub2api-gpt-5-5` 不再出现在用户默认选择里。
- Arena 的 `meta.access_grants` 含 `user:*:read`。

## 不要做什么

- 不要只改前端下拉文字。
- 不要只给管理员配置模型。
- 不要把 `arena-model` 当成普通 `model` 表记录处理完就结束。
- 不要让每次容器重启又把五档 preset 变成 inactive。
