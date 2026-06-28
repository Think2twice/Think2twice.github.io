# 10. 侧边栏分组标题要稳定可读

## 目标

让侧边栏里的“模型 / 分组 / 对话”等分区标题在深浅色模式下更清楚，不因为上游样式变化显得忽大忽小。

## 这次改动解决什么问题

侧边栏是每天都会反复扫视的区域。分区标题如果颜色太浅、字号不稳，会让“模型、分组、对话”的层级感变弱。

这次只做轻量运行时增强：

- 找到侧边栏里可见的短分区标题。
- 给它们加一个稳定 class。
- 注入一段小 CSS 控制字号、行高、字重和深色模式颜色。

## 手工复现流程

1. 新增 sidebar polish 脚本：

   ```text
   static/__fay/openwebui-sidebar-polish.js
   ```

2. 在 `src/app.html` 加载它：

   ```html
   <script defer src="/__fay/openwebui-sidebar-polish.js?v=20260625a"></script>
   ```

3. 用 `MutationObserver` 监听侧栏变化。

   OpenWebUI 侧栏会随路由、搜索和模型列表刷新，脚本不能只跑一次。

## 验证门禁

- `node --check static/__fay/openwebui-sidebar-polish.js`
- 切换深浅色模式，分区标题仍清楚。
- 路由切换或侧栏重新渲染后，分区标题仍保留样式。
- 不影响普通聊天条目、按钮和模型名称。

## 不要做什么

- 不要把所有短文本都强行改样式，必须限制在左侧可见窄区域。
- 不要把 sidebar polish 和刷新缓存、快捷键绑定放在同一个提交。
