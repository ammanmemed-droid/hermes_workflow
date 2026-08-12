开发习惯：用 Hermes 调度编码代理跑四阶段流水线（Codex 出方案 W1 → Claude Code 写代码 W2 → Codex 审查 W3 → Claude Code 测试 W4），带双看板进度跟踪（项目内 docs/PROGRESS.md + hermes kanban 按项目建 board，按阶段粒度更新），每轮结束做项目级 skills 沉淀（<项目名>-dev）；铁律：流水线全程不自动 git commit，等用户自己验证功能 OK 并明确说"提交"后才提交
§
Codex CLI：npm 全局 0.147.0 稳定版（PATH 内直接 `codex`，自带 code-mode-host 组件）；旧 alpha（~/.codex/.sandbox-bin/codex.exe）缺 code-mode-host 会导致 exec/review 报 "failed to spawn code-mode host"，已弃用；auth 已配置（~/.codex/auth.json）
§
Claude Code CLI 2.1.x npm 全局安装在 PATH；auth 走 ANTHROPIC_AUTH_TOKEN，base URL 指向 https://api.deepseek.com/anthropic（DeepSeek Anthropic 兼容端点）
§
工作流备份仓库：https://github.com/ammanmemed-droid/hermes_workflow.git（本地克隆 ~/hermes_workflow，含 skills/memories/kanban）；换机器后从这恢复；更新备份 = 重跑 cp 三个目录 + git push
§
本机 git 推 GitHub 偶发 443 连接超时（瞬时，亚太 IP 20.205.243.166 直连可达），重试或加 -c http.lowSpeedLimit=0 -c http.lowSpeedTime=999 可解
§
流水线偏好：角色-工具映射可灵活切换，用户口头说一句即覆盖默认（临时本轮/长期改默认均可，如"这轮 W2 用 codex"）；推理档位由 Hermes 按任务复杂度自动选择，用户可用"高推理/省钱模式"等口头覆盖；注意 Claude Code 经 DeepSeek Anthropic 端点时 --effort/--model 未必生效，深度思考优先靠提示词层面控制
§
用户用昵称指 Codex 模型：'5.6sol' = gpt-5.6-sol；未知昵称解析法：grep ~/.codex/sessions/**/*.jsonl 的 model 字段看历史会话真实模型名；本机 codex 亦走 DeepSeek Anthropic 端点（config.toml shell_environment_policy.set，但会话元数据 model_provider 记录为 openai）
§
Hermes 会话会给子进程注入 PYTHONPATH（hermes-agent + 其 venv site-packages），跑项目 Python 测试/脚本必须 env -u PYTHONPATH 或 PYTHONPATH= 前缀，否则依赖会解析到 Hermes venv 报 pydantic_core 等 ModuleNotFoundError
§
用户自定义技能库在 E:\wsf-skills\（含 backend-project-delivery：L1/L2/L3 分级交付路由器+升级确认门，定位是给 Codex 的风险/范围路由器，附 AGENTS.backend-routing.md 模板）；尚未装入 Hermes 技能库
§
工作流分级偏好：L1 机械小改（prefix/配置值/内部名/无生产影响 Nacos 名）由 Hermes 直接快改，1-3 分钟，不启多代理/设计/全量回归；小 Bug、小功能、少量业务逻辑优先 L2-Lite（简短方案人工确认→单代理开发→针对性验证），不默认四阶段；复杂/影响较广才 L2-Full 四阶段；L3 先授权。关键确认门：W1→开发、开发中超范围/实质取舍、审查阻断修复方案、验收后明确“提交”；不得自主决定或自动 commit。