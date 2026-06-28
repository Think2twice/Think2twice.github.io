# 12. Workspace seeds 要能重复安装和回滚

## 目标

把常用 Prompts、Skills、Tools 和 Action Functions 做成可版本管理的 seed 包，而不是只存在于某一次数据库状态里。

## 这次改动解决什么问题

OpenWebUI 的工作空间资源很容易变成“点出来的配置”：

- 换机器或换服务后不知道怎么恢复。
- Prompt/Skill/Tool 改过什么没有记录。
- Action function 全局启用会污染所有模型和所有对话。

新的 workspace seeds 采用文件化管理：

- `workspace-seeds/prompts/`：可复用提示词。
- `workspace-seeds/skills/`：角色和能力模板。
- `workspace-seeds/tools/`：普通工具函数。
- `workspace-seeds/functions/`：Action functions，默认不全局污染。
- `install-workspace-seeds.py`：通过 OpenWebUI API 安装、更新、验证或卸载。

## 手工复现流程

1. 把每类资源按目录落盘：

   ```text
   fay-custom/workspace-seeds/
   ```

2. Markdown seed 使用 frontmatter：

   ```markdown
   ---
   id: fay-study-coach
   name: Fay Study Coach
   tags: study, fay
   ---
   正文内容
   ```

3. Python 工具/函数用 docstring 写元信息：

   ```python
   """
   title: Clean Markdown
   description: Clean up markdown formatting.
   active: true
   global: false
   """
   ```

4. 先做本地校验：

   ```bash
   python3 fay-custom/scripts/install-workspace-seeds.py --validate-only
   ```

5. 安装时用 token 或账号密码：

   ```bash
   OPENWEBUI_TOKEN=... python3 fay-custom/scripts/install-workspace-seeds.py --base-url http://127.0.0.1:8080
   ```

6. 如果要回滚，只删除 seed 包管理的资源：

   ```bash
   python3 fay-custom/scripts/install-workspace-seeds.py --uninstall --dry-run
   ```

## 验证门禁

- `python3 -m py_compile fay-custom/scripts/install-workspace-seeds.py`
- `python3 fay-custom/scripts/install-workspace-seeds.py --validate-only`
- seed 文件不包含真实 token、账号密码、私密服务器信息。
- Action functions 默认 `global: false`，只有明确需要时才全局启用。

## 不要做什么

- 不要直接改数据库当作长期方案。
- 不要让 action functions 默认全局污染所有聊天。
- 不要把 token 写进 seed 文件或教程。
