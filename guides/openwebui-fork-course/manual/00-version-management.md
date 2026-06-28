# 版本管理：私有 fork 的正确提交栈

## 目标

把已经混在一个工作树里的 OpenWebUI 定制，整理成一条能阅读、能回滚、能 cherry-pick 的私有提交栈。

## 对应提交

095a0c8d9 / 402fb1c5b / 7642da192

## 为什么这一节重要

- 这不是简单的“补几个 commit”。当功能已经跨过前端、后端、运行配置、部署模板和公开教程时，版本管理本身就变成产品治理的一部分。一个大提交会让以后无法判断哪次改动引入了 Safari 问题、哪次改动影响了文件附件、哪次改动只是教程更新。
- 这个项目采用 stacked commits：每个提交都是前一个提交之上的增量，完整分支代表当前产品。它适合私有 fork，因为 Fay 需要自己维护一套完整体验；如果以后某个小功能适合贡献给上游，再从 upstream 新开干净分支，单独 cherry-pick 那一两个提交。
- 长期保留一个功能一个 branch 听起来清楚，但在这个项目里会变成维护负担。模型分级、品牌、移动端、工具路由、部署模板都会共享同一批配置和组件。并行长期分支越多，冲突越多，最后反而不知道哪条线是“真的产品”。

## 关键文件地图

- `fay-custom/references/commit-stack-plan.md`：记录提交拆分原则、当前栈和剩余本地文件。
- `fay-custom/tutorials/course/`：每个功能对应一节课程。
- `fay-custom/tutorials/manual/`：每个功能对应手工复现流程。
- `backup/fay-split-before-20260628-224120`：拆分前安全点。

## 手工复现流程

1. 先创建备份分支，保证拆分失败时能回到原始大工作树。
2. 用 `git status --short` 和 `git diff --stat` 把改动按产品原因分组，而不是按文件扩展名分组。
3. 每组先 staged，再跑该组自己的验证门禁。门禁通过后 commit，然后立即 push 到私有 fork。
4. 每个提交都写一节对应教程，避免以后只看到代码而忘了当初为什么这么改。
5. 拆完后记录最终提交栈，并保留未确认来源的本地临时文件，不顺手提交。

## 常用命令骨架

```bash
git status --short
git add fay-custom/references/commit-stack-plan.md
git diff --cached
git diff --cached --check
git commit -m "docs(fay): 版本管理"
git push origin codex/fay-openwebui-custom
```

## 知识课

- `upstream` 是官方 OpenWebUI，`origin` 是 Fay 自己的 private fork；这两个 remote 的身份不要混。
- commit 是项目经验的最小叙事单位。好的 commit 不是越碎越好，而是每个提交都有一个清晰的产品理由。
- 私有 fork 可以比 PR 分支承载更多定制，但仍然需要让历史可阅读。否则 fork 变成一次性拷贝，后续升级上游会很痛。
- 文档和代码同提交，能把“怎么修”和“为什么修”绑在一起；教程单独晚写，很容易丢掉当时的判断过程。

## 验证门禁

- `gh repo view ... --json visibility,isPrivate` 确认 fork 是 private。
- `git log --oneline --decorate` 确认本地 HEAD 和远程分支一致。
- 每个功能提交前运行对应的语法、HTML、敏感信息和 diff 检查。
- 最终只剩确认过不提交的本地临时文件。

## 常见坑

- 不要为了显得“功能很多”硬拆成无法独立解释的提交。
- 不要把每个功能都做成长期分支，除非它真的准备独立发 PR。
- 不要把公开教程和私有部署凭据混在一个页面里。
- 不要在脏的 Pages 仓库里直接发布，先停下来确认无关改动。

## 练习

拿任意一个已有提交，问自己三句话：这个提交解决了什么用户问题？它能不能单独回滚？如果我要给上游提 PR，它是否足够干净？

## 连续阅读

- 上一节：无
- 下一节：01. 本地开发启动底座：先让源码可运行
