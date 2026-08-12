---
name: openspec
description: "Use when user wants OpenSpec spec-driven change workflow."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [OpenSpec, Spec-Driven, Changes, Proposals, Requirements]
    related_skills: [backend-project-delivery, openspec-propose, openspec-apply-change, openspec-explore, openspec-update-change, openspec-sync-specs, openspec-archive-change]
---

# OpenSpec 规范驱动开发

OpenSpec 是 AI 原生规范驱动开发系统（CLI 包 `@fission-ai/openspec`，本机已装 1.8.0）。它把需求变成可追踪的变更产物：`proposal.md`（做什么/为什么）→ `design.md`（怎么做）→ `tasks.md`（实施步骤），实现后 `verify` 校验、`sync` 合并回主规范、`archive` 归档。

## 触发条件（不自动触发）

仅在这些情况下使用，不能因为任务复杂或已安装本技能就自动启用：

- 用户明确要求正式规格、变更提案或需求治理；
- 或已确认的契约、迁移、跨团队变更需要可追踪规格。

L3 不自动等于 OpenSpec。`backend-project-delivery` 仍是风险与范围的上游路由器；本技能只补充规范工作流，不扩大 BPD 的范围上限。

## 项目初始化（首次）

```bash
openspec init --tools hermes --no-animation
```

- 在项目生成 `openspec/config.yaml` 与 `.hermes/skills/` 下的 6 个官方 OpenSpec Hermes 技能（openspec-propose / apply-change / explore / update-change / sync-specs / archive-change）。
- 若 Hermes 未自动识别项目级 `.hermes/skills/`，将官方技能复制到 Hermes 技能库（本次已装到 `skills/openspec/`）。
- 项目内中文产出：在 `openspec/config.yaml` 设置 `context`，模板见 `templates/config.zh.yaml`。

## 命令速查（slash 命令 / CLI 对应）

| 阶段 | Slash 命令 | 作用 |
|---|---|---|
| 教程 | `/opsx:onboard` | 交互式完整流程入门教程 |
| 需求调研 | `/opsx:explore [话题]` | 提交变更前思考想法、调查问题、澄清需求（产出草稿，存 `openspec/explorations/`） |
| 建变更+全产物 | `/opsx:propose [变更名或描述]` | 一步生成 `changes/<变更名>/` 的 proposal.md、design.md、tasks.md |
| 建脚手架 | `/opsx:new [变更名]` | 只建变更文件夹，等后续命令生成产物 |
| 续产物 | `/opsx:continue [变更名]` | 按依赖链逐个生成（proposal → design → tasks） |
| 快进产物 | `/opsx:ff [变更名]` | 一次生成所有规划产物 |
| 编码实现 | `/opsx:apply [变更名]` | 实现变更中的任务 |
| 更新产物 | `/opsx:update [变更名]` | 修改计划产物并保持一致（实验性） |
| 质量校验 | `/opsx:verify [变更名]` | 提交前校验实现与变更产物匹配 |
| 合并主规范 | `/opsx:sync [变更名]` | 增量规范合并到主规范 |
| 归档 | `/opsx:archive [变更名]` | 归档完成的变更 |
| 批量归档 | `/opsx:bulk-archive [变更名...]` | 批量归档，自动处理规范冲突 |

CLI 等价命令：`openspec new change <name>`、`openspec status --change <name> --json`、`openspec instructions <artifact>`、`openspec apply`、`openspec validate <name>`、`openspec archive <name>`、`openspec list`、`openspec view`。

## 标准工作流

1. **规划边界**：propose/explore 只产出规划产物，不写项目代码；产物完成后停下，等用户发起 apply。
2. **实现**：apply 严格按 tasks.md 执行，改动范围内不越界；每个任务完成即验证。
3. **校验**：verify 对照 proposal/design 检查实现；未满足门槛报告“受阻”，不虚构成功。
4. **合并与归档**：sync 合并增量规范，archive 归档变更；两者都确认主规范无冲突。
5. **人工确认门**：方案产物需用户审核后才能进入实现；实现中遇到范围扩大、契约变化或技术取舍暂停询问；提交 Git 必须等用户明确说“提交”。

## 与分级流程的关系

- L1 快速修改不使用 OpenSpec。
- L2-Lite/L2-Full 仅在用户要求正式规格或变更需要可追踪产物时，将 OpenSpec 作为规划工具接入对应阶段。
- L3 中只有用户明确要求规格治理，或契约/迁移/跨团队变更确认需要追踪时使用；L3 授权不自动等于 OpenSpec 授权。

## 验证

- `openspec validate [item-name]` 校验变更与规范；
- `openspec status --change <name> --json` 查看产物完成度；
- `openspec doctor` 报告 OpenSpec 根目录关系健康度；
- `openspec list` / `openspec view` 查看变更与规范状态。
