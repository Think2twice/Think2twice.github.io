# iOS Safari/PWA 单击与输入修复

## 目标

解决 iPhone Safari 里按钮要双击、输入框点不进去、侧栏遮罩不能点空白关闭等移动端交互问题。

## 对应提交

324482199

## 为什么这一节重要

- iOS Safari 的问题不能按桌面浏览器经验处理。很多时候第一下 tap 被 hover、focus、tooltip、preload 或 touch/click 顺序吃掉，用户看到的就是“每个按钮都要点两次”。
- 本项目不是一个单 bug：从 preload hover、Tippy tooltip、iOS-only tap bridge、ProseMirror 输入框聚焦，到移动侧栏遮罩收起，都是同一类“触摸设备事件边界”问题。
- 正确做法不是给某个按钮硬绑 click，而是分层修：先让触摸设备不创建会吞首击的 tooltip，再针对 iOS WebKit 加 bridge，再对富文本编辑器做组件级 focus。

## 关键文件地图

- `src/app.html`：`data-sveltekit-preload-data` 从 hover 调整到 tap。
- `src/lib/components/common/Tooltip.svelte`：触摸设备不创建 Tippy 实例。
- `static/__fay/openwebui-ios-tap-bridge.js`：iOS WebKit 首击桥接。
- `src/lib/components/chat/MessageInput.svelte`：输入区域触摸聚焦编辑器。
- `src/lib/components/layout/Sidebar.svelte`：移动端遮罩显式关闭。

## 手工复现流程

1. 先区分桌面正常、安卓正常、iOS Safari 异常，确认不是业务按钮本身坏了。
2. 把 hover preload 调整成 tap，减少首击被预加载链路占用。
3. 触摸环境下禁用 tooltip 实例，避免第一下 tap 只触发 tooltip。
4. 只在 iOS/iPadOS WebKit 上启用 tap bridge，并跳过输入框、编辑器、地图等区域。
5. 对 Tiptap/ProseMirror 输入框，在容器 touchstart capture 阶段主动 focus 编辑器。
6. 侧栏遮罩用明确 close 函数，不用 toggle，避免点空白区变成状态不确定。

## 常用命令骨架

```bash
git status --short
git add src/app.html
git diff --cached
git diff --cached --check
git commit -m "feat(fay): Safari 修复"
git push origin codex/fay-openwebui-custom
```

## 知识课

- 移动端点击不是简单的 click；touchstart、touchend、pointer、focus、click 的顺序会影响最终行为。
- Tooltip 类库常用 hover/focus 触发，在触摸设备上容易吞掉首击。
- 富文本编辑器有自己的 DOM 和 selection 状态，只 focus 外层容器不一定能弹键盘。
- 遮罩关闭应该是 idempotent 的 close，不应该是 toggle。

## 验证门禁

- iOS 触摸环境中 Tippy 实例数量为 0。
- 单次 tap 按钮能打开面板，不需要第二次。
- 点击输入区域出现插入光标并弹出键盘。
- 侧栏打开后点右侧遮罩任意空白处收起。

## 常见坑

- 不要只在 desktop Chrome 验收移动端 bug。
- 不要把所有 touchend 都强制 click，输入框和编辑器会被破坏。
- 不要用 toggle 处理遮罩关闭。
- 不要把用户实机复现当作缓存问题草草跳过。

## 练习

拿一个移动端按钮，画出 touchstart -> touchend -> click -> focus 的事件顺序，再想想 tooltip 插在中间会发生什么。

## 连续阅读

- 上一节：04. ChatGPT 品牌壳：不是只换一个图标
- 下一节：06. 文件生成：必须变成真实附件
