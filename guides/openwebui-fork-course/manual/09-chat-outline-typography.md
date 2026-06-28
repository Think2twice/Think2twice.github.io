# 09. 长对话要有用户问题大纲

## 目标

给长对话增加右侧用户问题大纲，并收紧聊天正文标题排版，让 OpenWebUI 更适合复盘和学习。

## 这次改动解决什么问题

长对话里真正的结构通常不是助手回答里的标题，而是用户一轮一轮问了什么。没有大纲时，复盘只能靠滚动条猜位置。

新的阅读体验做两件事：

- 从用户消息生成右侧大纲，点击可跳转到对应轮次。
- 收紧 `h1/h2/h3` 的字号和间距，让聊天正文更像工作文档，而不是营销页。

## 手工复现流程

1. 新增 `ChatOutline.svelte`。

   ```text
   src/lib/components/chat/ChatOutline.svelte
   ```

   它从 `messages` 里筛出用户消息，清理 Markdown、图片和代码块后生成短标题。

2. 在 `Messages.svelte` 的消息列表旁挂载大纲。

   ```text
   src/lib/components/chat/Messages.svelte
   ```

   大纲只在桌面宽屏和至少两条用户消息时显示，避免移动端挤占空间。

3. 调整聊天正文标题 CSS。

   ```text
   static/__fay/openwebui-chat-typography.css
   ```

   重点是减少标题过大造成的视觉断裂，保持阅读节奏稳定。

## 验证门禁

- Svelte parse：`ChatOutline.svelte` 和 `Messages.svelte`
- `node --check` 不适用于 CSS，但要检查 CSS 无明显拼写错误。
- 长对话里右侧出现短横线；hover 展开显示用户问题。
- 点击任意大纲项能滚动到对应用户消息。
- 移动端不显示右侧大纲，不遮挡输入框。

## 不要做什么

- 不要把助手回答标题当作主导航，它们不一定代表用户复盘线索。
- 不要让大纲在移动端常驻。
- 不要用视口宽度动态缩放字体，标题应使用稳定字号。
