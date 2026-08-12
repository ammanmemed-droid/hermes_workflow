开发习惯：用 Hermes 调度编码代理跑四阶段流水线（Codex 出方案 W1 → Claude Code 写代码 W2 → Codex 审查 W3 → Claude Code 测试 W4），带双看板进度跟踪（项目内 docs/PROGRESS.md + hermes kanban 按项目建 board，按阶段粒度更新），每轮结束做项目级 skills 沉淀（<项目名>-dev）；铁律：流水线全程不自动 git commit，等用户自己验证功能 OK 并明确说"提交"后才提交
§
Codex CLI 装在 C:\Users\shifeng.wang\.codex\.sandbox-bin\codex.exe（0.147.0-alpha），不在 PATH；auth 已配置（~/.codex/auth.json）；git-bash 里用全路径调用
§
Claude Code CLI 2.1.x npm 全局安装在 PATH；auth 走 ANTHROPIC_AUTH_TOKEN，base URL 指向 https://api.deepseek.com/anthropic（DeepSeek Anthropic 兼容端点）