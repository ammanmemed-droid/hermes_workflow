# hermes_workflow

Hermes Agent 工作流资产备份（跨机器迁移用）。

## 目录结构

```
skills/     # Hermes 技能（含分级路由器 + 可裁剪 codex-pipeline）
memories/   # 跨会话记忆（用户偏好、环境事实）
kanban/     # 进度看板数据（SQLite boards）
codex/      # Codex 全局 AGENTS.md、backend-project-delivery 技能、superpowers 插件包
```

## 核心工作流：分级交付 + codex-pipeline

先由 `backend-project-delivery` 分级：L1 快速修改、L2-Lite 单代理轻量开发、L2-Full 四阶段、L3 高风险授权流程。只有 L2-Full/L3 默认使用完整 Codex 方案 → Claude 开发 → Codex 审查 → Claude 测试；方案和审查修复均需人工确认。**不自动 git commit**。

按需专项技能（不自动触发）：
- `superpowers/`：brainstorming、writing-plans、systematic-debugging、TDD 等流程技能（v6.2.0，来自 superpowers-marketplace；与 Hermes 内置重名技能共存，保留现有版本）。
- `openspec/`：OpenSpec 规范驱动开发（CLI 1.8.0），仅用户要求正式规格/契约/迁移追踪时启用。

详见 `skills/autonomous-ai-agents/codex-pipeline/SKILL.md`

## 新机器恢复步骤

👉 **完整换机手册见 [docs/MIGRATION.md](docs/MIGRATION.md)**（10 分钟恢复）

速览：
1. 安装 Hermes：`curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`
2. 安装编码 CLI 并登录（Codex：`npm i -g @openai/codex` + login；Claude Code：`npm i -g @anthropic-ai/claude-code` + ANTHROPIC_AUTH_TOKEN 指向 DeepSeek 端点）
3. `git clone https://github.com/ammanmemed-droid/hermes_workflow.git`
4. 拷贝 `skills/ memories/ kanban/` 到 Hermes home；拷贝 `codex/AGENTS.md` 和 `codex/skills/` 到 `~/.codex/`
5. 重启 Hermes；后端需求会先自动分级，也可说「用 codex-pipeline 跑 <需求>」

## 安全说明

`.gitignore` 排除了 auth.json / .env / *.lock 等敏感文件。Codex 与 Claude 的登录凭证**不在此仓库**，换机器需重新登录。
