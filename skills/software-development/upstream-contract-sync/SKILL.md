---
name: upstream-contract-sync
description: "Sync local tool schemas when upstream API contracts change."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [contract, schema, upstream, api, sync, middleware, tool-platform]
    related_skills: [roxie-management-platform-dev]
---

# 上游 API 契约变更对齐（中台 schema 同步）

上游服务（RAG 知识库、第三方 API 等）升级接口契约时，中台/网关/代理服务的本地 schema 会失配。典型症状：调用返回 `ToolOutputValidationError` / `code=50001` / "Additional properties are not allowed" / "X is a required property"。

## When to use

- 上游说"接口字段改了/加了字段/字段改名了"
- 中台校验报 `additionalProperties` 或 `required` 相关错误
- 上游接口文档版本号变化（如 0.8.0 → 0.8.1）

## 标准流程

1. **确认上游实际版本**：拉上游 `openapi.json` 的 `info.version`（文档可能过时，以运行时为准）
2. **拉实时 schema**：上游若提供 `GET /api/v1/tools/{tool_name}` 之类的详情接口，直接拿实时 input/output schema（比本地静态文件权威）
3. **对比本地 vs 远程**：逐对象对比 `properties`/`required` 差异，列出变化清单（顶层 + 子对象 $defs）
4. **分类处理**（见下）
5. **若有 sync 脚本**：先改脚本 `expected_version` 为上游实际版本，再跑脚本全量重拉（比手动改更稳，且顺带吸收其他遗漏差异）
6. **同步测试夹具**：契约测试若从文档/示例文件读取响应 JSON（如 `examples/orchestrator/README.md` 的 ```json block），必须同步补上新 required 字段，否则测试失败
7. **全量回归**：跑完整测试确认无其他工具被误伤

## 变化分类处理规则

| 上游变化 | 中台动作 |
|---|---|
| 新增**合法字段**（如 `troubleshooting`、`predictions`） | 必须跟进 schema（`additionalProperties: false` 会拒绝未知字段） |
| 返回**契约外多余字段**（上游 bug，如临时加的 `xxx_source`） | 中台不用改，等上游修；上游下个版本移除后自动恢复 |
| **单工具字段改名**（如 `shared_parts` → `parts`） | 只改该工具的 schema，其他工具不动 |
| 入参 `required` 变化 | 以运行时 schema 为准；sync 会覆盖，不要手动加回旧 required |

## Pitfalls

1. **枚举值 ≠ 字段名**：改名时只改字段，别碰枚举值（如 `group_reason` 枚举里的 `shared_parts`/`same_system_shared_parts`）和展示文本（如 `grouping_basis` 里的 `shared_parts=...`）——它们不是字段
2. **字段可选性**：上游"未知时省略"的字段设为可选（不进 required），缺失不影响校验
3. **文档示例与测试联动**：README/示例文件里的响应 JSON 缺失新 required 字段 → 契约测试直接挂；测试通常按 ```json block 顺序取（如 index*2+1 是响应），补字段位置要对
4. **上游不可达时**：以文档 + 用户提供的真实响应为准做手动对比，不要猜字段类型；用户贴的真实响应可拿去直接跑 `validate_against_schema` 验证
5. **区分"上游 bug"与"合法变更"**：拿不准时问用户（用户通常知道上游意图）；不要为了通过校验就放宽 `additionalProperties`

## 验证

- 用用户提供的真实响应跑 `validate_against_schema`（strip_invalid_nulls 后）确认通过
- 全量测试通过
- 若有 sync 脚本，重跑一次确认幂等（schema 不再变化）

## 参考

- `references/roxie-rag-081.md` — roxie 项目 RAG 0.8.1 对齐实操记录（脚本、URL、字段清单、踩坑）
