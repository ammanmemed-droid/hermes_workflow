---
name: codex-pipeline
description: "Codex方案→Claude开发→Codex审查→Claude测试 + 看板/skills沉淀."
version: 1.2.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [Codex, Claude-Code, Orchestration, Pipeline, Kanban, Code-Review, Testing, Skills]
    related_skills: [codex, claude-code, hermes-agent]
---

# Codex + Claude Code 混合四阶段流水线（带进度看板）

用 Hermes 调度两个编码代理 CLI，按「Codex 方案 → Claude Code 开发 → Codex 审查 → Claude Code 测试」四阶段流水线完成开发，全程维护**双看板进度跟踪**（项目内 PROGRESS.md + hermes kanban），每轮结束做项目级 skills 沉淀。Hermes 是总调度：起窗口、传话、更新看板、收尾沉淀。

## When to use

- 用户要求按多窗口流水线开发，混合使用 codex 和 claude code
- 用户提到「codex 输出方案」「claude code 写代码」「codex 审查」「claude code 测试」「进度看板」「待办」「已完成」
- 流水线：方案制定、开发、审查、测试分窗口，带进度跟踪

## Prerequisites

- **Codex CLI**（本机：`~/.codex/.sandbox-bin/codex.exe`，**不在 PATH**，全路径调用）
- **Claude Code CLI**（本机：npm 全局 `claude` 2.1.x，在 PATH；auth 走 `ANTHROPIC_AUTH_TOKEN` → `https://api.deepseek.com/anthropic` 兼容端点）
- **hermes kanban**（`hermes kanban init` 一次性初始化；看板按项目分 board）
- 目标目录必须是 git 仓库
- codex 调用必须 `pty=true`；claude 用 `-p` 打印模式不需要 pty

## ⛔ Git 提交铁律（用户明确要求）

**流水线全程绝不自动 `git commit`** —— 包括 PROGRESS.md 和所有代码产物。代理窗口的产出留在工作区，等**用户自己验证功能没问题后，用户明确说"提交"**，才执行 commit（用户验证通过后通常一次性提交代码 + PROGRESS.md）。

## 双看板进度跟踪（每阶段必做）

### A. 项目内看板 `docs/PROGRESS.md`
每阶段开始/结束由 Hermes 更新，模板：

```markdown
# <项目名> 开发看板

## 📊 当前状态
- 当前阶段：W3 代码审查（第 2 轮）｜ 进度：3/5 ｜ 更新：<时间>

## ✅ 已完成
- [x] W1 方案制定（第1轮）→ docs/design.md
- [x] W2 代码开发（第1轮）→ commit 待用户确认

## 🔄 进行中
- [ ] W2 代码开发（第2轮）：<模块>

## 📋 待办
- [ ] W3 代码审查（第2轮）
- [ ] W4 测试（第2轮）
- [ ] 第2轮技能沉淀
```

更新时机（**按阶段粒度**，不用拆任务）：
- 阶段启动 → 「待办→进行中」，刷新当前阶段/更新时间
- 阶段完成（产出验证通过后）→ 「进行中→已完成」+ 一行产出摘要（产物路径/测试数/commit 状态）
- 每轮结束 → 追加「技能沉淀」完成项
- 用户可在预览面板 `open_preview docs/PROGRESS.md` 实时查看

### B. hermes kanban 后端（机器可查）
```bash
hermes kanban init                    # 一次性
hermes kanban boards create <slug>    # 每个项目一个 board（如 repo 名）
hermes kanban boards switch <slug>    # 切到项目板
hermes kanban create "W1 方案制定 - <需求>" --body "<要点>" --workspace dir:<repo绝对路径>
hermes kanban complete t_xxx          # 阶段完成
hermes kanban comment t_xxx "产出：docs/design.md，3 模块"   # 留产出记录
hermes kanban list                    # 查看
```
- 每阶段 = 一张卡（W1..W4 + 技能沉淀），阶段启动时 create（状态 ready），完成后 complete
- 产出要点用 comment 留在卡上；`hermes kanban show t_xxx` 看历史
- 状态机：todo → ready → running → review → blocked → done

## Pipeline 五步

### 0. 启动前准备
- `git status` 确认工作区干净，脏则先 stash
- 初始化看板：写 `docs/PROGRESS.md` 模板；`hermes kanban boards create <slug>` + switch（首次）
- 约定产物：设计文档 `docs/design.md`、审查报告 `docs/review.md`、看板 `docs/PROGRESS.md`
- 每个窗口独立后台会话：`terminal(command=..., workdir=<repo>, background=true, pty=true)`（claude -p 可不带 pty）；长任务 `notify_on_complete=true`

### 1. W1 方案制定 — Codex
```bash
~/.codex/.sandbox-bin/codex.exe exec --sandbox workspace-write \
  "分析需求：<需求描述>。产出 docs/design.md：含目标、技术选型、模块拆分、接口设计、任务分解、验收标准。不要写业务代码。"
```
- 启动：看板 A/B 同步「W1 进行中」
- 完成：读 docs/design.md 验证 → 看板「W1 已完成」→ 要点给用户确认后进 W2

### 2. W2 代码开发 — Claude Code
```bash
claude -p "按照 docs/design.md 实现<模块/功能>。遵循项目现有代码风格。实现完成**不要 commit**，停在工作区。" \
  --allowedTools "Read,Edit,Write,Bash(git*),Bash(python*),Bash(pip*),Bash(npm*),Bash(uv*)" --max-turns 30
```
- 启动/完成同 W1 更新看板；完成后 `git status`/`git diff --stat` 验证改动在但**未提交**

### 3. W3 代码审查 — Codex
```bash
~/.codex/.sandbox-bin/codex.exe exec \
  "审查工作区未提交改动（git diff），输出 docs/review.md：问题分级（阻断/建议/风格）、文件+行号+修改建议。"
```
- 阻断问题 → 回灌 W2：新 claude -p 窗口按 review 意见改 → 回 W3 复审直到通过；阻断期看板标 blocked

### 4. W4 测试 — Claude Code
- 先由 Hermes 直接跑项目测试命令（快、省 token）；失败或补用例再交给 claude
```bash
claude -p "运行测试（<测试命令>），修复失败测试或补缺失用例，确保全部通过。不要 commit。" \
  --allowedTools "Read,Edit,Write,Bash" --max-turns 20
```
- 验证：重跑测试全绿 → 看板 W4 完成

### 5. 技能沉淀 + 进度简报（每轮必做）
- 看板 A/B 同步「技能沉淀」完成项
- `skill_manage` 创建/更新项目专属技能（`<repo名>-dev`，category 用项目名）：项目结构、构建/测试命令、风格约定、架构决策与理由、踩坑（→Pitfalls）、用户纠正（→立即写入）
- 给用户**进度简报**：本轮各阶段产出、看板现状、下一步待办
- **不 commit**，提醒用户自行验证，验证通过后说一声再提交

## 并行与隔离

- 四阶段**默认串行**；同一仓库多个代理窗口改同批文件会冲突 → `git worktree` 隔离或严格串行
- 独立无依赖任务（多 issue）才并行：worktree + 每窗口一个后台会话

## Pitfalls

1. **codex 不在 PATH**（Windows）：必须 `~/.codex/.sandbox-bin/codex.exe`；alias：`alias codex=~/.codex/.sandbox-bin/codex.exe`
2. **codex 必须 pty=true**；claude -p 不要加 pty
3. **codex sandbox**：开发用 `--sandbox workspace-write`；gateway/service 环境 bubblewrap 报错改用 `--sandbox danger-full-access`
4. **claude -p 必带 `--max-turns` 和 `--allowedTools`**；交互式多轮才用 tmux（信任弹窗 Enter，--dangerously-skip-permissions 弹窗 Down+Enter）
5. **绝不要自动 commit**（用户铁律）：代理提示词里明确写"不要 commit"；阶段验证用 `git status`/`git diff` 而非 git log
6. **hermes kanban**：`boards rm` 不支持 `--yes`（直接 rm 即归档）；`boards switch` 切板；任务 id 形如 t_xxxxxxxx；completed 任务 list 显示 ✓
7. **验证自报**：代理说完成≠完成，必须读文件/跑测试核实
8. **窗口饥饿**：逐个 wait 完成再起下一个
9. **技能沉淀 + 看板更新别跳过**：用户工作流核心环节，每阶段必做

## Verification

- W1：docs/design.md 存在且完整；看板 W1 done
- W2：`git status` 有改动且**未提交**；看板 W2 done
- W3：docs/review.md 存在，无阻断级问题；看板 W3 done
- W4：测试命令全绿；看板 W4 done
- 看板：PROGRESS.md 与 hermes kanban 状态一致，均可查看
- 沉淀：skill_manage 成功，skill_view 可读；进度简报已给用户
- **git 状态：工作区有未提交改动，等用户验证后指令提交**
