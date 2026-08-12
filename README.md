# hermes_workflow

Hermes Agent 工作流资产备份（跨机器迁移用）。

## 目录结构

```
skills/     # 全部技能（含 codex-pipeline 四阶段流水线 + 项目专属技能）
memories/   # 跨会话记忆（用户偏好、环境事实）
kanban/     # 进度看板数据（SQLite boards）
```

## 核心工作流：codex-pipeline

Codex 方案(W1) → Claude Code 开发(W2) → Codex 审查(W3) → Claude Code 测试(W4) → skills 沉淀
全程双看板：项目内 `docs/PROGRESS.md` + hermes kanban；**不自动 git commit**（用户验证后手动提交）。

详见 `skills/autonomous-ai-agents/codex-pipeline/SKILL.md`

## 新机器恢复步骤

👉 **完整换机手册见 [docs/MIGRATION.md](docs/MIGRATION.md)**（10 分钟恢复）

速览：
1. 安装 Hermes：`curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`
2. 安装编码 CLI 并登录（Codex：`npm i -g @openai/codex` + login；Claude Code：`npm i -g @anthropic-ai/claude-code` + ANTHROPIC_AUTH_TOKEN 指向 DeepSeek 端点）
3. `git clone https://github.com/ammanmemed-droid/hermes_workflow.git`
4. 拷贝 `skills/ memories/ kanban/` 到 `$LOCALAPPDATA/hermes/`（Windows）或 `~/.hermes/`（macOS/Linux）
5. 重启 Hermes，说「用 codex-pipeline 跑 <需求>」

## 安全说明

`.gitignore` 排除了 auth.json / .env / *.lock 等敏感文件。Codex 与 Claude 的登录凭证**不在此仓库**，换机器需重新登录。
