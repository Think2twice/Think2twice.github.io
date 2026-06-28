# 00. Version management for Fay's OpenWebUI fork

## Goal

Turn a large working tree into a readable private Git history without losing
the ability to push the whole product or extract a small upstream pull request
later.

## Correct branch model

Use a private stacked integration branch:

```bash
git checkout codex/fay-openwebui-custom
```

Each feature becomes one logical commit on that branch. The last commit contains
all previous work because commits form a stack.

Do not maintain one long-lived branch per feature for this project. Many Fay
features share files such as `src/app.html`, `middleware.py`, `ResponseMessage`,
and deployment templates. Independent branches would create repeated conflicts
and make the fork harder to understand.

## How to push

Push the stacked branch to the private fork:

```bash
git push origin codex/fay-openwebui-custom
```

This publishes the whole private product line. GitHub will show each commit in
order.

## How to extract one upstream PR later

Create a clean branch from upstream and cherry-pick the relevant commit:

```bash
git fetch upstream main
git checkout -b upstream-pr/mobile-safari-tap upstream/main
git cherry-pick <commit-sha>
```

If the commit depends on earlier infrastructure commits, either cherry-pick that
small dependency first or split the original commit further before proposing it
upstream.

## Safety workflow before splitting commits

1. Confirm the source repo:

   ```bash
   pwd
   git status --short
   git remote -v
   ```

2. Confirm the fork is private:

   ```bash
   gh repo view Think2twice/open-webui --json nameWithOwner,visibility,isPrivate,url
   ```

3. Create a safety branch:

   ```bash
   git branch backup/fay-split-before-YYYYMMDD-HHMMSS
   ```

4. Stage one feature at a time:

   ```bash
   git add path/to/file
   git add -p path/with/mixed-changes
   git diff --cached
   git commit -m "feat(scope): concise summary"
   ```

5. Push after a coherent batch:

   ```bash
   git push origin codex/fay-openwebui-custom
   ```

## What not to commit

- `.env`, keys, tokens, cookies, private local runtime state.
- `output/` screenshots and generated probe files.
- Server-only hotfixes unless they are also reflected in source or deployment
  templates.

Evidence belongs in `PROJECT_PROGRESS.md`; raw probe files can stay local.
