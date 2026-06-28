# Workspace seeds：资源包要能重复安装和回滚

## 目标

把 Prompts、Skills、Tools、Action Functions 做成可重复安装的默认资源包，而不是让 Fay 在 UI 里一个个手工添加。

## 对应提交

34eac1f93

## 为什么这一节重要

- OpenWebUI 的工作区能力很多，但如果每个提示词、技能、工具都靠手工点 UI 添加，项目很快不可复制。换机器、重建数据库、部署服务器时，资源就会丢或者版本不一致。
- 种子包的关键不是“塞很多东西”，而是可控：哪些资源默认给普通用户读、哪些工具有执行风险、Action 是否全局显示、外部 OpenAPI/MCP 是否默认启用，都要有边界。
- 本项目里 Action Functions 一开始全局挂载导致每条回复下面出现一排图标，这就是资源安装没有权限和 UI 策略的后果。

## 关键文件地图

- `fay-custom/workspace-seeds/`：Prompts、Skills、Tools、Functions 源文件。
- `fay-custom/scripts/install-workspace-seeds.py`：通过 OpenWebUI API upsert。
- `fay-custom/workspace-seeds/tool-servers.example.json`：默认关闭的外部工具候选。
- `fay-custom/LESSONS.md`：Action 默认不能全局挂载的规则。

## 手工复现流程

1. 先把资源分类：Prompt、Skill、Tool、Action Function、外部工具服务器。
2. 每个资源用文件表达，避免只存在于数据库。
3. 安装器走 API upsert，不直接改生产数据库。
4. Prompts/Skills/Tools 给 `user:*:read`，Functions 默认为 `active=true, global=false`。
5. 先跑 `--validate-only`，再跑 `--dry-run`，最后才真实安装。
6. 安装前备份数据库，安装后用 API 或 DB 摘要验数量和权限。

## 常用命令骨架

```bash
git status --short
git add fay-custom/workspace-seeds/
git diff --cached
git diff --cached --check
git commit -m "feat(fay): 工作区种子"
git push origin codex/fay-openwebui-custom
```

## 知识课

- 种子包是应用配置即代码的一种形式。
- 工具和函数有执行风险，不能像提示词一样无脑公开。
- API upsert 比直接写 DB 更尊重应用自己的校验和迁移逻辑。
- 默认关闭外部工具服务器，是为了避免上下文外发和额度风险。

## 验证门禁

- validate-only 能解析所有资源。
- dry-run 显示将创建或更新哪些条目。
- 安装后数量符合预期。
- Functions 的 `is_global=0`，不会污染每条消息动作栏。

## 常见坑

- 不要把 Tools/Functions 当成普通文档资源批量公开。
- 不要直接改生产 DB 而不走 API。
- 不要让 Action 默认全局挂载。
- 不要启用外部工具服务器而不做白名单和审计。

## 练习

给一个团队设计 5 个默认 Prompt、3 个 Skill、2 个 Tool。写出谁能看、谁能执行、如何回滚。

## 连续阅读

- 上一节：11. 刷新快捷键：页面刷新不能等于重新生成
- 下一节：13. 全源码部署：服务器覆盖模板要收口
