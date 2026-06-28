# 刷新快捷键：页面刷新不能等于重新生成

## 目标

修复 Command+R / Command+Shift+R 被聊天应用误当成重新生成的问题，并让聊天详情读取走 network-first。

## 对应提交

090455bb7

## 为什么这一节重要

- 用户按 Command+R 的心智是“刷新页面”，不是“让模型重新回答一次”。如果浏览器快捷键被应用层抢掉，尤其在 Mac 上就会造成很强的不安全感：我只是刷新，为什么又消耗了一次请求？
- 聊天详情缓存也会放大这个问题。页面刷新后如果先吃旧缓存，用户会看到第四个问题一会儿消失、一会儿回来，以为数据丢了。network-first 的意义是优先问服务器当前真实状态，再用缓存兜底。
- 这节课的本质是快捷键和数据一致性。用户熟悉的系统快捷键要尊重；应用自己的重新生成应该换成更明确、不容易误触的组合。

## 关键文件地图

- `src/routes/(app)/+layout.svelte`：全局快捷键处理。
- `src/lib/utils/shortcuts.ts`：快捷键定义和平台判断。
- `static/__fay/openwebui-chat-cache.js`：聊天详情 network-first 补丁。
- `fay-custom/tutorials/manual/11-refresh-shortcuts-cache.md`：手工流程记录。

## 手工复现流程

1. 列出浏览器系统快捷键：刷新、强刷、关闭标签、搜索等，应用层不要抢。
2. 把重新生成改到更明确的组合，比如 `Mod+Alt+R`。
3. 聊天详情读取改成 network-first：先拉服务器，失败再用缓存。
4. 刷新后检查问题列表和消息详情是否一致。
5. 把这个规则写进教程，因为它是桌面 Web App 的基本礼貌。

## 常用命令骨架

```bash
git status --short
git add src/routes/(app)/+layout.svelte
git diff --cached
git diff --cached --check
git commit -m "feat(fay): 刷新快捷键"
git push origin codex/fay-openwebui-custom
```

## 知识课

- Web 应用能监听键盘，但不能假装自己比浏览器更懂系统快捷键。
- 缓存策略分 cache-first、network-first、stale-while-revalidate，不同页面适合不同策略。
- 聊天消息是强一致感很高的数据，用户刷新时更想看到服务器真实状态。
- 重新生成是高风险动作，应该有明确按钮或不易误触的快捷键。

## 验证门禁

- Command+R 触发浏览器刷新，不触发重新生成。
- Command+Shift+R 仍是浏览器强刷语义。
- 重新生成快捷键变为明确组合。
- 刷新后最新问题不会短暂消失。

## 常见坑

- 不要把浏览器刷新快捷键占为应用快捷键。
- 不要让 cache-first 管住聊天主数据。
- 不要用“偶尔刷新回来”当作数据一致性没问题。
- 不要让重新生成这种动作没有清晰触发边界。

## 练习

列出你常用的 10 个浏览器快捷键，标出哪些应用绝对不应该抢，哪些可以在编辑器焦点内覆盖。

## 连续阅读

- 上一节：10. 侧边栏分组标题：稳定、可读、不抢戏
- 下一节：12. Workspace seeds：资源包要能重复安装和回滚
