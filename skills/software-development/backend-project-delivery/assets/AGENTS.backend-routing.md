# 后端任务 Skill 路由

- 任何后端仓库的开发、修改、调试、审计、优化或交付任务，默认先应用 `backend-project-delivery` 判断 L1/L2/L3；用户不需要显式写 `$backend-project-delivery`。
- L1 局部机械修改只走快速通道，不因“修改了文件”而自动启用 OpenSpec、需求设计、完整计划或 TDD。
- L2 Bug 使用系统化调试和针对性 TDD；L2 新功能或业务行为变化按需使用需求澄清、设计和 TDD。
- L3 先取得升级确认；正式规格、契约、迁移或跨团队治理需要可追踪变更时再使用 OpenSpec。
- Codex 内置或其他领域 Skill 只补充任务直接需要的知识与工具，不改变 `backend-project-delivery` 已确定的范围和验证上限。
- 平台强制规则、用户明确指令或更具体的仓库指令优先；存在冲突时先说明，不静默扩大流程。
