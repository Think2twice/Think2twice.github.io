# 06. 文件生成必须变成真实附件

## 目标

让 DOCX、PDF、PPTX、XLSX、CSV、Markdown、HTML、ZIP 等生成结果成为 OpenWebUI 里的真实附件，而不是只在聊天正文里说“已生成”。

## 这次改动解决什么问题

旧链路有三个常见失败：

- Python 在 Pyodide 里确实写了文件，但前端没有把文件上传回 OpenWebUI。
- 后端 fallback PDF 只是简陋占位，中文长文、标题、列表和代码块排版很差。
- 前端把过程代码、stdout、stderr 混在正文里，用户看不到干净结果，也不一定能下载附件。

新的文件生成链路要求：

- Pyodide 执行后扫描 `/mnt/uploads`，把新增文件作为 `File` 上传。
- 后端 `generate_file` 工具生成文件后写入 OpenWebUI Files 表，并关联当前 chat message。
- PDF 走 OpenWebUI 的 `fpdf2` + NotoSans/NotoSansSC 字体栈。
- 前端显示真实附件，过程信息默认折叠到状态卡里。
- 文件卡片点击时优先触发浏览器下载，而不是只打开预览。

## 手工复现流程

1. 在 Pyodide worker 里记录执行前后的 `/mnt/uploads` 文件差异。

2. 把新增文件通过 `uploadFile()` 上传回 OpenWebUI：

   ```text
   src/lib/workers/pyodide.worker.ts
   src/routes/+layout.svelte
   ```

3. 后端新增通用 PDF 生成器：

   ```text
   backend/open_webui/utils/pdf_generator.py
   ```

   它用 OpenWebUI 已带的字体生成可读 PDF，避免中文断字、缺字和纯占位 PDF。

4. 后端内置工具注册 `generate_file`：

   ```text
   backend/open_webui/tools/builtin.py
   backend/open_webui/utils/tools.py
   ```

5. 前端渲染层合并 `message.files`、code-interpreter output 和 details 里的文件信息。

6. 把过程内容挂到状态卡展开区，完成态正文只保留一句干净提示。

## 验证门禁

- `python -m py_compile backend/open_webui/utils/pdf_generator.py backend/open_webui/tools/builtin.py backend/open_webui/utils/tools.py`
- `node --check` 或 TypeScript 转译 `src/lib/workers/pyodide.worker.ts`
- Svelte parse：`ResponseMessage.svelte`、`StatusHistory.svelte`、`FileItem.svelte`
- 生成 DOCX/PDF/XLSX/PPTX/ZIP 时，消息上方有真实附件。
- 下载接口返回附件头和正确文件签名。
- 完成后正文不显示大段 Python 原文，过程卡默认折叠。

## 不要做什么

- 不要只靠模型文字说“文件已生成”。
- 不要把只有 `%PDF` 签名的占位 PDF 当完成。
- 不要把 stdout/stderr 当作用户正文。
- 不要只支持一个格式，用户要求多格式时要保留多个附件。
