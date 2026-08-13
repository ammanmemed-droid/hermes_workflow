---
name: codex-pipeline
description: "Tiered Codex+Claude dev pipeline: plan→code→review→test."
version: 2.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [Codex, Claude-Code, Orchestration, Pipeline, Kanban, Review, Testing]
    related_skills: [backend-project-delivery, codex, claude-code, hermes-agent]
---

# 分级 Codex + Claude Code 开发流水线

Hermes 先使用 `backend-project-delivery` 判断风险与范围，再选择最小必要流程。BPD 是分级权威；本 Skill 只负责编排执行，不得因多代理、Superpowers、OpenSpec 或其他 Skill 而扩大任务等级、修改范围或验证范围。

## 优先级

平台强制规则 > 用户当前明确指令 > 当前作用域仓库指令 > BPD 分级和范围上限 > 本流水线 > Superpowers/OpenSpec/TDD/调试及领域 Skill。存在冲突时说明冲突，不静默执行重流程。

## 四档路由

| 档位 | 判断 | 默认执行 |
|---|---|---|
| L1 | 局部、机械、可逆，不改变业务/数据/权限语义 | Hermes 直接精准搜索、最小修改、旧值检查、diff、最多一个快速检查；不启动流水线 |
| L2-Lite | 小 Bug、小功能或少量业务逻辑；范围明确、复杂度低、通常少量文件 | 简短方案 → 用户确认 → 单代理开发 → Hermes diff 与针对性测试 → 用户验收 |
| L2-Full | 复杂逻辑、影响较广、跨核心模块或独立审查价值明显 | W1 Codex → 用户确认 → W2 Claude → W3 Codex → 修复确认门 → W4 Claude |
| L3 | 生产服务发现、公开契约、安全/权限、数据迁移、不可逆操作 | 先取得升级授权，再按风险执行完整分析、兼容、验证和分段授权 |

任务是 Bug、新功能或跨模块修改，并不自动等于 L2-Full。优先选择 L2-Lite；只有证据支持时才建议 Full。

## L1 快速通道

严格遵循 `backend-project-delivery` 的 L1 契约。禁止设计文档、完整计划、看板、需求矩阵、新增测试、全量回归、完整启动、多代理窗口和 Skill 沉淀。发现契约、多个消费者、生产服务发现、安全、数据或范围扩大信号时停止，合并为一次升级确认请求。

## L2-Lite

1. **简短方案：** 当前行为、目标行为、预计文件、验证范围、风险和非目标；在聊天中输出即可。
2. **Gate 1：** 用户确认方案后才开发。
3. **单代理开发：** 默认按任务选择 Hermes、Codex 或 Claude Code，只实施确认范围，不 commit。
4. **过程确认：** 设计前提不成立、需改方案外文件、存在实质技术取舍、依赖/契约/Schema/权限/生产配置变化时立即暂停询问。
5. **验证：** Hermes 检查 diff，运行目标行为/目标模块的针对性测试；无证据不跑全仓回归。
6. **交付：** 给用户行为变化、文件、测试和剩余风险，等待验收与“提交”。

默认不建双看板、不单独开启审查窗口、不强制项目 Skill 沉淀。任务持续较长或遗漏风险明显时才启用看板。

## L2-Full

### W1 Codex 方案

使用 PATH 中 npm 稳定版 `codex`（当前 0.147.0），不要使用旧 `~/.codex/.sandbox-bin/codex.exe` alpha。方案包含目标/非目标、当前与目标行为、文件范围、决策、风险、验证和验收标准。默认简短；确需正式设计时才使用 Superpowers。OpenSpec 仅在用户要求正式规格/提案/治理，或已确认的契约、迁移、跨团队变更需要追踪时使用。

**Gate 1：方案完成后停止。必须由用户审核确认，才能移交 W2。**

### W2 Claude Code 开发

Claude Code只实施已确认方案，不 commit。开发中出现以下任一情况立即暂停：超范围文件、前提失效、多个实质方案、兼容策略、依赖升级、Schema/数据/权限、生产配置、删除或扩大重构、用户改动重叠。

确认请求必须给出：证据、原方案、选项、推荐、影响；确认前不代替用户决定。

### W3 Codex 审查

审查未提交 diff，报告阻断/建议/风格及文件行号。代理自报必须由 Hermes 核实。

**修复确认门：** 有阻断时先向用户提交问题证据和修复方案；用户确认后才回灌 W2。不得自动修复。修复后再复审。

### W4 Claude Code 测试

按证据选择目标行为、目标模块和必要直接消费者测试；无证据不默认全仓回归。Hermes 必须重跑关键命令验证。Python 项目测试清理 Hermes 注入的 `PYTHONPATH`：

```bash
env -u PYTHONPATH .venv/Scripts/python.exe -m pytest <target>
# 或
PYTHONPATH= uv run pytest <target>
```

准确标记证据等级：Mock/Fake、进程内 HTTP、契约 Stub、共享环境、类生产/生产。不得把低等级证据写成集成通过或可发布。

## L3 与分段授权

先使用 BPD 升级确认门。用户确认后按需加载服务发现、安全与数据、AI 系统等专项参考。只读分析、代码/配置修改、部署/灰度、不可逆生产操作分别授权；前一阶段授权不自动延伸。未满足外部门槛时报告“受阻”或 `integration_unverified`，不写“有条件通过”。

## Superpowers 与 OpenSpec

- L1 不调用需求设计、完整计划、TDD、Superpowers 或 OpenSpec。
- L2 只在任务确实匹配时按需使用 brainstorming、系统化调试、TDD、设计或计划执行。
- OpenSpec 只在用户明确要求，或已确认的契约/迁移/跨团队变更需要可追踪规格时使用；L3 不自动等于 OpenSpec。
- 专项 Skill 只补充方法，不能扩大 BPD 的范围上限。

## 看板策略

- L1：不建看板。
- L2-Lite：默认不建；持续较长或遗漏风险明显时使用简化 `docs/PROGRESS.md`。
- L2-Full/L3：使用项目 `docs/PROGRESS.md` + Hermes Kanban，按阶段更新，不拆成细碎任务。
- 人工 Gate 状态必须标记为 blocked/等待用户确认。

## Git 铁律

全程不自动 commit、stash、reset、checkout 或覆盖用户改动。工作区不干净时先判断是否与目标重叠；不重叠则保留，重叠则询问。只有用户自己验收并明确说“提交”后才 commit。

## 项目 Skill 沉淀

L1 不沉淀。L2-Lite 默认不沉淀。L2-Full/L3 只有产生稳定、可复用的项目知识、命令、架构约定或非平凡 Pitfall 时才创建/更新 `<repo>-dev`；不得保存凭据、内部敏感内容或短期进度。

## 完成检查

- 分级有证据，流程没有超过允许上限。
- W1 方案与审查修复均通过人工确认门。
- 开发过程中的范围/技术决策没有被代理自主决定。
- diff 聚焦且未覆盖用户工作。
- 测试范围与风险匹配，结果由 Hermes 实际核实。
- 工作区保持未提交，等待用户验收。
