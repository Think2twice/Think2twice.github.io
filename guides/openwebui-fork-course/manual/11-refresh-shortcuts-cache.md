# 11. 刷新快捷键不能触发重新生成

## 目标

让浏览器刷新键回到浏览器本身：`Command+R` 和 `Command+Shift+R` 应该刷新页面，不应该触发 OpenWebUI 的“重新生成回答”。

## 这次改动解决什么问题

旧行为里，`Command+R` 被应用快捷键注册成重新生成回答。对 Mac 用户来说，这和浏览器刷新习惯冲突很大：

- 想刷新页面，结果重新生成回答。
- `Command+Shift+R` 也可能因为快捷键匹配不严格被误吃掉。
- 聊天缓存如果优先返回旧缓存，刷新后还可能看到旧的消息列表。

新的规则：

- “重新生成回答”改成 `Command+Option+R`。
- 快捷键匹配时严格区分是否按下 Shift。
- 浏览器刷新组合键直接放行。
- 聊天详情 GET 请求改成 network-first，只有网络失败才回退缓存。

## 手工复现流程

1. 改快捷键注册：

   ```text
   src/lib/shortcuts.ts
   ```

   `Regenerate Response` 从 `mod + R` 改为 `mod + alt + R`。

2. 改全局快捷键匹配：

   ```text
   src/routes/(app)/+layout.svelte
   ```

   如果快捷键没有声明 Shift，就不能被 `Command+Shift+R` 命中。

3. 放行浏览器刷新键：

   ```text
   isBrowserRefreshShortcut(event)
   ```

   `Command/Ctrl + R` 且没有 Alt 时直接 return，不阻止浏览器默认刷新。

4. 改聊天缓存策略：

   ```text
   static/__fay/openwebui-chat-cache.js
   ```

   GET 聊天详情先请求网络，网络失败时再用 IndexedDB/local memory 缓存兜底。

5. 在 `src/app.html` 更新脚本版本号。

## 验证门禁

- Svelte parse：`src/routes/(app)/+layout.svelte`
- TypeScript transpile：`src/lib/shortcuts.ts`
- `node --check static/__fay/openwebui-chat-cache.js`
- Mac 上按 `Command+R`：浏览器刷新，不重新生成回答。
- 按 `Command+Shift+R`：浏览器强刷，不重新生成回答。
- 按 `Command+Option+R`：才触发重新生成。
- 刷新后优先拿网络数据，断网时才用缓存兜底。

## 不要做什么

- 不要把浏览器标准刷新快捷键绑定到应用动作。
- 不要为了缓存速度牺牲刷新后的正确性。
- 不要把刷新修复和 sidebar/笔记/部署配置混在一起。
