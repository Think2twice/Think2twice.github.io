# 文件生成：必须变成真实附件

## 目标

让 DOCX、PDF、PPTX、XLSX、ZIP 等生成结果以 OpenWebUI 真实附件形式出现，而不是正文里的假路径或空白页。

## 对应提交

4027e9673

## 为什么这一节重要

- 用户要的是“文件已经附在这条回复下面，可以点开下载”，不是看到一段 Python 代码、`sandbox:` 假链接或 `/mnt/uploads` 路径。文件生成的 UX 成败取决于四层：生成、注册、下载、前端展示。
- 这类功能不能只看服务器上有没有文件。过去的问题就是文件存在，但 UI 里点开空白页、正文过程摊开、PDF 质量差、只生成一个格式而不是用户要求的多个格式。
- 官方 ChatGPT 的体验是结果优先、过程可展开。这个项目也采用同样原则：可见正文干净，附件是真实按钮，过程默认折叠，用户想看再展开。

## 关键文件地图

- `backend/open_webui/tools/builtin.py`：文件生成和附件注册逻辑。
- `backend/open_webui/utils/pdf_generator.py`：结构化 PDF 生成。
- `src/lib/components/chat/Messages/ResponseMessage.svelte`：附件和过程卡片展示。
- `fay-custom/scripts/check-*`：文件生成和附件路由回归脚本。

## 手工复现流程

1. 先识别用户要的是文件产物，而不是图片或普通聊天。
2. 后端生成文件后，必须注册到 OpenWebUI Files，拿到真实 file id。
3. 助手消息的 `files` 字段要包含真实附件元数据，前端才能渲染附件按钮。
4. 正文清洗掉代码、stdout、sandbox 假路径，只保留简洁结果提示。
5. 过程信息进入折叠卡片，包含执行代码、stdout/stderr、注册结果和文件列表。
6. PDF 不能只保证 `%PDF`，还要检查中文字体、标题层级、代码块和长文排版。

## 常用命令骨架

```bash
git status --short
git add backend/open_webui/tools/builtin.py
git diff --cached
git diff --cached --check
git commit -m "feat(fay): 文件附件"
git push origin codex/fay-openwebui-custom
```

## 知识课

- Artifact 是一等结果，不是聊天正文里的链接文本。
- 下载接口要返回 attachment header、正确字节数和正确文件签名。
- 前端按钮定位要点真实附件 chip，不能只按文件名找第一个文本。
- PDF 生成需要字体栈和换行策略，中文长文不能用最简占位 PDF。

## 验证门禁

- 生成文件存在且注册到 Files。
- 认证下载接口返回真实附件而不是前端 HTML。
- 浏览器点击附件按钮触发 download 事件。
- 最新助手消息不出现 `sandbox:`、`<details>`、裸 Python 过程等坏词。
- PDF 打开后能看到标题、列表、代码块和中文正文。

## 常见坑

- 不要把文件写到磁盘就算完成。
- 不要把正文里的路径当成下载入口。
- 不要用脚本点错目标后误判 UI 坏了。
- 不要把“PDF 合法”当成“PDF 可读”。

## 练习

用四层表检查一个文件功能：生成层、注册层、下载层、UI 层。每层写一个失败现象和一个验证命令。

## 连续阅读

- 上一节：05. iOS Safari/PWA 单击与输入修复
- 下一节：07. 图片生成和编辑：先判定意图，再调用工具
