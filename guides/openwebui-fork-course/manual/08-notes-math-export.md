# 笔记捕获、数学渲染和导出

## 目标

让回答能稳定添加到笔记，公式能正确渲染，导出 PDF/Markdown 时保持可读排版。

## 对应提交

2b656ebcd

## 为什么这一节重要

- 笔记功能不是“把文本 append 到一个字段”那么简单。OpenWebUI 的 note 结构里 Markdown、HTML、JSON 位置不同；如果按猜测写入，就会出现看似成功但内容为空、公式重复、导出缺样式等问题。
- Fay 的使用方式是把长回答沉淀成可复习材料，所以笔记必须支持数学公式、代码块、表格和导出。否则它只是聊天收藏，不是学习资料系统。
- 导出体验也要像正式产品：菜单语义清楚，下载动作真实发生，导出的 PDF 不应该丢公式、丢代码块或出现重复标题。

## 关键文件地图

- `src/lib/components/notes/`：笔记编辑、显示和导出入口。
- `src/lib/components/chat/Messages/ResponseMessage.svelte`：添加到笔记按钮。
- `src/lib/utils/note-math.ts`：公式渲染和源文本保留。
- `static/__fay/openwebui-note-export.js`：导出增强脚本。

## 手工复现流程

1. 先用 API 抓一份真实 note detail，确认内容在 `data.content.md/html/json` 的哪一层。
2. 添加笔记时保留 Markdown 源，HTML 渲染只作为显示层，不覆盖源内容。
3. 公式渲染时给原始公式加 data 属性，避免导出时丢失或重复渲染。
4. 导出菜单按用户语言命名，PDF、Markdown、HTML 等动作明确分开。
5. 验证不要只看截图，还要检查 API 内容、DOM `.katex`、导出文件字节和可读性。

## 常用命令骨架

```bash
git status --short
git add src/lib/components/notes/
git diff --cached
git diff --cached --check
git commit -m "feat(fay): 笔记导出"
git push origin codex/fay-openwebui-custom
```

## 知识课

- 富文本系统通常有源文本层和渲染层，二者不能互相覆盖。
- 数学公式渲染要考虑 inline/block、转义、重复渲染和导出降级。
- 导出不是复制 DOM，需要处理样式、分页、代码块和表格。
- 真实数据结构优先于猜测字段名。

## 验证门禁

- 点击添加后，最近笔记的 `data.content.md` 含回答内容。
- 页面 DOM 中出现 `.katex`，同时保留原始公式 source。
- 导出 PDF 菜单能触发下载。
- 导出文件包含标题、正文、公式、代码块和表格。

## 常见坑

- 不要按字段名猜 note 结构。
- 不要只看视觉截图，忽略 API 持久化。
- 不要把公式渲染后的 HTML 当作唯一源。
- 不要让导出按钮变成预览按钮。

## 练习

给一个含公式、表格、代码块的回答设计笔记验收表：API 内容、页面 DOM、导出 PDF、导出 Markdown 各验什么。

## 连续阅读

- 上一节：07. 图片生成和编辑：先判定意图，再调用工具
- 下一节：09. 长对话大纲与排版：用户问题才是目录
