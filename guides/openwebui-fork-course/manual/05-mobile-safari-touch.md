# 05. iOS Safari/PWA 单击与输入修复

## 目标

修复 Fay 在 iPhone Safari 和主屏幕 PWA 里遇到的三类问题：

- 按钮、菜单、工具入口经常要点两次。
- 聊天输入框点不出光标和键盘。
- 移动端侧栏打开后，点右侧遮罩空白区不能收起。

## 根因分层

这不是一个按钮坏了，而是移动 Safari 的触摸事件、hover/focus、SvelteKit preload、Tooltip 和编辑器焦点共同作用。

本次修复分四层：

- `src/app.css`：触摸设备上给按钮、链接、菜单等交互元素加 `touch-action: manipulation`。
- `src/app.html`：把 SvelteKit 预加载从 `hover` 改为 `tap`，并加载 iOS-only tap bridge。
- `Tooltip.svelte`：触摸设备上不创建 Tippy tooltip，避免第一次 tap 被 tooltip hover/focus 消耗。
- `MessageInput.svelte`：点聊天输入容器时，在 `touchstart` capture 阶段直接 focus Tiptap/ProseMirror 编辑器。
- `Sidebar.svelte`：移动端遮罩不再 toggle，而是明确 `showSidebar.set(false)`。

## 手工复现流程

1. 改 `src/app.html`：

   ```html
   <body data-sveltekit-preload-data="tap">
   <script src="/__fay/openwebui-ios-tap-bridge.js?v=20260628a"></script>
   ```

2. 新增 `static/__fay/openwebui-ios-tap-bridge.js`。

   这个脚本只在 iOS/iPadOS WebKit 上启用。它在 `touchend` capture 阶段找到可点击目标并触发一次真实 `click()`，随后抑制重复 native click。输入框、textarea、ProseMirror、终端、地图、toast 等区域会跳过。

3. 改 `Tooltip.svelte`：

   用 `navigator.maxTouchPoints`、`(hover: none)`、`(pointer: coarse)` 判断触摸环境。触摸环境直接 destroy tooltip，不再创建 Tippy。

4. 改 `MessageInput.svelte`：

   在 `#chat-input-container` 上加：

   ```svelte
   data-fay-ios-editor-focus="chat-input"
   on:touchstart|capture={focusChatInputFromTouch}
   ```

5. 改 `Sidebar.svelte`：

   移动端遮罩绑定 `pointerdown`、`touchstart`、`mousedown`、`click`，并用 `|self` 确保只点遮罩本体时关闭。

## 验证门禁

- `node --check static/__fay/openwebui-ios-tap-bridge.js`
- Svelte parse：`Tooltip.svelte`、`MessageInput.svelte`、`Sidebar.svelte`
- 首页 HTML 包含 `data-sveltekit-preload-data="tap"` 和 iOS bridge。
- iPhone Safari 直接访问：普通按钮单击触发。
- 主屏幕 PWA：点输入区域应出现光标和键盘。
- 移动端侧栏打开后，点右侧灰色遮罩应立即收起。

## 不要做什么

- 不要继续给单个按钮加特判。
- 不要让全局 bridge 抢输入框、编辑器、终端、地图的原生交互。
- 不要让遮罩继续 toggle，否则状态抖动时可能越点越乱。
