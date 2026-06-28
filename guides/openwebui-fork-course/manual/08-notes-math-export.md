# 08. 笔记捕获、数学渲染和导出要进源码

## 目标

把“添加到笔记”和笔记里的数学公式处理从运行时 DOM 补丁迁回 OpenWebUI 源码，让它可维护、可保存、可导出。

## 这次改动解决什么问题

旧方案靠 `static/__fay/openwebui-add-to-note.js` 在页面加载后扫描按钮区，再插入“添加到笔记”按钮。这类补丁有三个问题：

- Svelte 组件重渲染后按钮可能丢失。
- 公式渲染只存在于 DOM 里，保存、编辑、导出时容易退化。
- PDF/打印导出靠事后修补 HTML，遇到长公式、表格、代码块就不稳定。

新的方案把笔记能力放回三个源码层：

- 聊天消息组件：按钮、去重、创建今日笔记、追加 Markdown。
- Markdown 工具：统一解析 `$...$`、`$$...$$`、`\(...\)`、`\[...\]`。
- 富文本编辑器：把公式作为带源码属性的节点保存，导出时还能还原成 Markdown。

## 手工复现流程

1. 在 `ResponseMessage.svelte` 里加原生“添加到笔记”按钮。

   ```text
   src/lib/components/chat/Messages/ResponseMessage.svelte
   ```

   按钮直接调用 Notes API，写入当天 `YYYY-MM-DD` 笔记，并用隐藏 marker 防止重复追加同一条回答。

2. 新增统一的笔记 Markdown/KaTeX 工具。

   ```text
   src/lib/utils/marked/note-math.ts
   ```

   这层负责把 Markdown 公式渲染成 HTML，同时保留 `data-fay-katex-source`，方便编辑器和导出恢复原公式。

3. 让 NoteEditor 优先从 Markdown 渲染每日笔记。

   ```text
   src/lib/components/notes/NoteEditor.svelte
   ```

   这样公式不依赖旧的页面级脚本，打开笔记时就能稳定显示。

4. 扩展 RichTextInput 的公式节点和 Turndown 规则。

   ```text
   src/lib/components/common/RichTextInput.svelte
   ```

   公式进入编辑器后仍然可复制、可保存、可导出。

5. 保留 `openwebui-note-export.js` 作为导出增强层。

   它继续负责打印/PDF 布局，但不再承担“给聊天按钮区注入功能”的职责。

## 验证门禁

- Svelte parse：`ResponseMessage.svelte`、`RichTextInput.svelte`、`NoteEditor.svelte`
- TypeScript transpile：`src/lib/utils/marked/note-math.ts`
- `node --check static/__fay/openwebui-note-export.js`
- 在一条含公式的回答上点击“添加到笔记”，当天笔记出现 Markdown 内容。
- 再点一次同一条回答，应提示已经添加过，不重复写入。
- 打开笔记编辑器，公式能显示；导出 PDF/打印时排版不丢公式。

## 不要做什么

- 不要继续靠 DOM 扫描给聊天按钮区补功能。
- 不要只保存渲染后的 KaTeX HTML，必须保留原始公式源码。
- 不要把笔记提交和聊天缓存、sidebar、部署配置混在一起。
