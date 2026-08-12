---
name: hermes-state-backup
description: "Backup/migrate Hermes state: skills, memory, kanban → git."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [Hermes, Backup, Migration, Skills, Memory, Kanban, Git, Dotfiles]
    related_skills: [codex-pipeline, hermes-agent]
---

# Hermes 状态备份与跨机器迁移

把 Hermes 的工作流资产（技能 skills/、记忆 memories/、看板 kanban/）备份到 git 仓库，换电脑/重装后 5 分钟恢复。凭证（auth.json / API key）**绝不入仓库**，CLI 登录在新机器重新配置。

## When to use

- 用户要求备份/上传 Hermes 工作流、技能、记忆、看板
- 用户问"换电脑后工作流还能用吗 / 怎么迁移"
- 每轮开发结束后更新工作流备份

## 资产位置（Windows，HERMES_HOME = $LOCALAPPDATA/hermes）

| 资产 | 源路径 | 备份到 |
|---|---|---|
| 全部技能 | `$LOCALAPPDATA/hermes/skills/` | `<repo>/skills/` |
| 记忆 | `$LOCALAPPDATA/hermes/memories/MEMORY.md` | `<repo>/memories/` |
| 看板 | `$LOCALAPPDATA/hermes/kanban/`（boards/ + current） | `<repo>/kanban/` |

用户现有备份仓库：`https://github.com/ammanmemed-droid/hermes_workflow.git`（本地 `~/hermes_workflow`，main）。

## 备份步骤

```bash
cd ~/hermes_workflow
cp -r "$LOCALAPPDATA/hermes/skills" ./skills
mkdir -p memories && cp "$LOCALAPPDATA/hermes/memories/MEMORY.md" ./memories/
cp -r "$LOCALAPPDATA/hermes/kanban" ./kanban
git add -A && git commit -m "backup: <日期/轮次>"
git push origin main
```

GitHub 推送偶发 443 瞬时超时（直连可达时）：直接重试，或
`git -c http.lowSpeedLimit=0 -c http.lowSpeedTime=999 push origin main`。

## 新机器恢复

1. 装 Hermes：`curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`
2. 装编码 CLI + 登录（凭证不入仓库，必须重配）：Codex（`npm i -g @openai/codex`，`~/.codex/auth.json`）；Claude Code（`npm i -g @anthropic-ai/claude-code`，`ANTHROPIC_AUTH_TOKEN`）
3. `git clone <repo>` → `cp -r skills/ memories/ kanban/ "$LOCALAPPDATA/hermes/"`
4. 新会话说"用 codex-pipeline 跑 <需求>"即恢复完整工作流

## 官方同步途径（备选）

`hermes sync`（Skill Sync）：`hermes sync status` → `hermes sync enable <skill>` → `hermes sync now`。
**前提：登录 Nous Portal**（`hermes login`）；未登录时 `hermes sync status` 显示 `logged_in: false`、sync inert → 走 git 方案。

## .gitignore 要点与验证

- 必须排除：`auth.json`、`*.auth.json`、`.env`、`.env.*`、`*.env`、`*.lock`、`state.db*`、`sessions/`、`logs/`、`cache/`
- 坑：`.env` 模式只匹配恰好叫 `.env` 的文件，`secret.env` 这类后缀要 `*.env` 才能挡住（真实踩过）
- 记忆文件用 `!memories/MEMORY.md` 强制保留
- 验证用命令不靠肉眼：
  - `git check-ignore -q <path>` 逐条测规则（应忽略 vs 应保留各测几条）
  - `git ls-files | grep -cE 'auth\.json|\.env|\.lock|^sessions/|^logs/'` → 必须为 0
- 提交前跑一遍，确认暂存内容无敏感文件再 push

## Pitfalls

1. **凭证不进仓库**：auth.json/.env 排除后，换机器必须重新登录，README 里写明恢复步骤
2. **技能归属**：`hermes curator adopt <skill>` 前，agent 创建的技能也按 user-owned 对待——若需要后台 curator 维护，先让用户在会话里执行 adopt
3. **Windows 路径**：git-bash 里用 `$LOCALAPPDATA/hermes`（即 `C:\Users\<user>\AppData\Local\hermes`），不是 `~/.hermes`
4. **推送失败先重试**：瞬时 443 超时常见，重试或加 lowSpeed 参数，不要立刻下"网络不通"结论（先 `curl -sS -o /dev/null -w "%{http_code}" https://github.com` 验证连通性）
