# 07. 图片生成和图片编辑要走意图路由

## 目标

让“生成图片”和“编辑当前上传图片”在 OpenWebUI 里变成稳定、可解释、可恢复的产品链路，而不是靠模型偶然调用工具。

## 这次改动解决什么问题

图片功能最容易出错的地方不是模型会不会画图，而是“这一轮到底该不该画图”：

- 用户只是分析图片时，不应该自动触发图片生成。
- 用户上传图片并要求“改一下”时，应该把当前轮图片作为编辑输入。
- 生成结果应该挂到当前消息附件里，刷新页面后仍然能看到。
- 图片生成中要有明确状态，不要让用户以为页面卡住。

新的链路把图片处理拆成四层：

- 意图层：判断当前轮是否明确要求生成或编辑图片。
- 输入层：只取当前轮上传的图片，不偷拿历史对话里的旧图片。
- 工具层：把单张或多张图片按后端 API 需要的格式传给图片编辑接口。
- 展示层：把生成图归一化成 OpenWebUI 文件附件，并显示图片生成状态卡。

## 手工复现流程

1. 在消息预处理层增加图片意图判断。

   ```text
   backend/open_webui/utils/middleware.py
   ```

   重点不是堆关键词，而是区分“看图/解释图”和“生成/编辑图”两个动作。

2. 对上传图片做当前轮作用域限制。

   ```text
   current user message -> current files -> image edit payload
   ```

   这样用户问“这张图改成头像风格”时，只编辑刚上传的图；用户后续普通聊天时，不会被旧图误触发。

3. 在内置图片工具里补编辑提示保护。

   ```text
   backend/open_webui/tools/builtin.py
   ```

   编辑图时给模型明确边界：保留主体、构图和主要元素，只执行用户明确提出的修改。

4. 把生成图附件归一化。

   ```text
   backend/open_webui/utils/middleware.py
   backend/open_webui/tools/builtin.py
   ```

   图片可能以 URL、file id、content type、message files 等多种形式出现，渲染前要统一成 `{ type: "image", url, name }`。

5. 前端状态卡显示“图片正在生成”。

   ```text
   src/lib/components/chat/Messages/ResponseMessage/StatusHistory.svelte
   src/lib/components/chat/Messages/ResponseMessage/StatusHistory/ImageGenerationStatusCard.svelte
   ```

## 验证门禁

- `python -m py_compile backend/open_webui/tools/builtin.py backend/open_webui/utils/middleware.py backend/open_webui/routers/images.py`
- Svelte parse：`StatusHistory.svelte` 和 `ImageGenerationStatusCard.svelte`
- `python fay-custom/scripts/check-image-attachment-normalization.py`
- `python fay-custom/scripts/check-image-context-isolation.py`
- 问“解释这张图”时不生成新图。
- 问“把这张图改成头像风格”时只使用当前轮上传图片。
- 刷新页面后，生成图仍作为当前消息附件显示。

## 不要做什么

- 不要因为上下文里有图片，就自动调用图片工具。
- 不要把历史消息里的图片拿来编辑当前轮。
- 不要只在正文里插 Markdown 图片，必须同步成消息附件。
- 不要把图片提交和笔记、聊天缓存、部署配置混在一起。
