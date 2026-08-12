# 换电脑迁移手册

> Hermes + Codex/Claude Code 分级开发工作流的换机恢复指南
> 恢复耗时约 10 分钟。备份仓库：https://github.com/ammanmemed-droid/hermes_workflow.git

## 恢复步骤

### 第 1 步：装 Hermes

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

装完先别用，继续往下。

### 第 2 步：装 Git + Node.js

- **Git**：官网安装（自带 Credential Manager，clone 时登录一次 GitHub）
- **Node.js**：官网装 LTS 版（自带 npm）

### 第 3 步：装两个编码 CLI 并登录

```bash
# Codex（W1 方案 + W3 审查）
npm install -g @openai/codex
codex login          # 或把旧电脑 ~/.codex/auth.json 拷过来

# Claude Code（W2 开发 + W4 测试）
npm install -g @anthropic-ai/claude-code
# 配置 DeepSeek Anthropic 兼容端点：
export ANTHROPIC_AUTH_TOKEN="<你的 DeepSeek Key>"
export ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"
```

> 建议把上面两个 export 写进 shell 配置文件（`~/.bashrc`），免每次手动设。

### 第 4 步：拉取工作流资产（关键）

```bash
git clone https://github.com/ammanmemed-droid/hermes_workflow.git
cd hermes_workflow
cp -r skills/ memories/ kanban/ "$LOCALAPPDATA/hermes/"   # Windows
mkdir -p ~/.codex/skills
cp codex/AGENTS.md ~/.codex/AGENTS.md
cp -r codex/skills/* ~/.codex/skills/
# macOS/Linux 用：cp -r skills/ memories/ kanban/ ~/.hermes/
```

### 第 4.5 步：恢复 Superpowers 与 OpenSpec

```bash
# Superpowers 插件（Codex 侧）：把备份的插件包放回缓存
mkdir -p ~/.codex/plugins/cache
cp -r codex/plugins/superpowers-marketplace ~/.codex/plugins/cache/
# 然后在 ~/.codex/config.toml 启用 marketplace 与插件（凭据字段重新填写）：
#   [plugins."superpowers@superpowers-marketplace"]  enabled = true
#   [marketplaces.superpowers-marketplace]  last_updated = "..."

# Hermes 侧：备份的 skills/ 已含 superpowers/ 与 openspec/ 分类，无需额外操作

# OpenSpec CLI（全局）
npm install -g @fission-ai/openspec@latest
# 新项目初始化（生成 openspec/ + .hermes/skills 官方技能）：
openspec init --tools hermes --no-animation
```

### 第 5 步：重启 Hermes 并验证

1. 关闭并重新打开 Hermes（让技能/记忆生效）
2. 新会话输入：

> 「帮我修改 XX，项目在 D:\projects\xxx」

看到任务先被判断为 L1 / L2-Lite / L2-Full / L3；小改动不启动四阶段 = 恢复成功 ✅

## 注意事项

| 事项 | 说明 |
|---|---|
| **登录凭证不进仓库** | 仓库里只有技能/记忆/看板，没有 auth.json（.gitignore 已排除）。Codex / Claude 登录必须在新电脑重做（第 3 步） |
| **Codex 版本** | 使用 npm 稳定版并确保 `codex` 在 PATH；不要使用旧 `~/.codex/.sandbox-bin/codex.exe` alpha |
| **备份要保持新鲜** | 改了技能后对 Hermes 说「更新工作流备份」，即重跑 `cp` 三个目录 + `git push` |
| **旧项目看板** | kanban 历史已备份；换机后旧项目若继续开发，看板数据直接可用 |

## 备份更新命令（手动）

```bash
# 在旧电脑上执行（或用 Hermes 代劳）
cp -r "$LOCALAPPDATA/hermes/skills" ~/hermes_workflow/skills
cp -r "$LOCALAPPDATA/hermes/memories" ~/hermes_workflow/memories
cp -r "$LOCALAPPDATA/hermes/kanban" ~/hermes_workflow/kanban
cd ~/hermes_workflow && git add -A && git commit -m "backup update" && git push
```

## 疑难排查

| 症状 | 处理 |
|---|---|
| 新会话不认 codex-pipeline | 检查 `$LOCALAPPDATA/hermes/skills/autonomous-ai-agents/codex-pipeline/` 是否存在；不存在则重跑第 4 步 |
| 推送 GitHub 超时 | 亚太网络瞬时抖动，重试；或加 `-c http.lowSpeedLimit=0 -c http.lowSpeedTime=999` |
| claude 提示未登录 | 重跑第 3 步的 export + `claude auth status --text` 检查 |
| codex 不在 PATH | 重新执行 `npm install -g @openai/codex`，确认 `codex --version` 可用 |
