---
name: upstream-api-contract-sync
description: "Upstream API/contract upgrade: re-sync local tool schemas."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [api-contract, schema-sync, upstream, integration]
    related_skills: [roxie-management-platform-dev, backend-project-delivery]
---

# 上游 API 契约同步（Upstream API Contract Sync）

上游服务（如 roxie-rag-service）升级版本/改字段时，中台（tool 中台）需要同步本地契约。本文是通用工作流，roxie 项目的具体路径见"项目落地"节。

## 触发条件

- 用户贴出上游新版本 API 文档（如 TOOL_SERVICES_API.md 0.8.1）问"中台要不要改"
- 调用上游报 `ToolOutputValidationError`（出参校验失败）——往往是上游新增/删除字段导致
- 上游发布说明提到字段、required、枚举变化

## 核心步骤

1. **先读文档，再对比本地**：提取上游文档的字段/required/枚举，与本地 `tool-schema.json` 对比（用 python 脚本批量打印顶层 properties 和 $defs 子字段，不要肉眼翻 JSON）。
2. **拉实时 schema 为准**：文档可能与运行时不一致，权威来源是上游的 `GET /api/v1/tools/{tool_name}`（或 openapi.json）。RAG 服务可达时直接 curl 对比——文档说 0.8.1 但服务还在 0.8.0，或文档漏了字段，都以实时为准。
3. **优先用同步脚本全量重拉**：项目若有 sync 脚本（如 `sync_rag_five_tool_contracts.py`），改版本号 `expected_version` 后直接跑，5 个 schema 一次性对齐——比手动逐个改 5 个 JSON 文件稳得多，也不容易漏。
4. **同步联调示例**：如果测试用文档/README 里的示例响应校验 output_schema（如 `test_orchestrator_examples.py` 读 `examples/orchestrator/README.md` 的 JSON block），**必须同步更新示例**，否则全量测试挂。示例要补新 required 字段（用真实响应数据补）。
5. **全量测试验证**：`env -u PYTHONPATH .venv/Scripts/python.exe -m pytest -q`（Hermes 会话不清 PYTHONPATH 会 ModuleNotFoundError）。

## 关键坑

1. **`additionalProperties: false` 拒绝未知字段**：上游新增字段而本地 schema 没加时，出参校验直接失败（ToolOutputValidationError, code=50001, HTTP 500）。用户贴的报错 message 会明确列出"Additional properties are not allowed ('xxx' were unexpected)"——照字段名加进对应 $defs 即可。上游**删除**字段则本地不用动（契约本来就比上游严）。
2. **required 要对齐**：上游把某字段从必填改可选（如 model），本地 input_schema required 也要跟着改，否则中台比上游严格，Agent 省略该字段会被中台 40000 拒绝（但上游能接受）。
3. **新 required 字段导致示例测试挂**：schema 更新后，示例响应缺新 required 字段 → `test_orchestrator_examples.py` 报 `(root): 'xxx' is a required property`。错误信息里带路径（如 `predictions.0: 'troubleshooting' is a required property`），照路径补示例。
4. **版本号硬校验**：sync 脚本 `expected_version` 不更新会直接 RuntimeError 拒绝跑（`expected RAG 0.8.1, got 0.8.0` 或反向）。升级契约第一步就是改这个。
5. **schema 字段对比用脚本**：`python -c "import json; ..."` 打印顶层 properties + $defs 子字段 + required，比 grep JSON 快且不会漏。

## 项目落地（roxie_management_platform）

- 五工具契约：`myskills/<skill>/tool-schema.json`（dtc-context/grouping/cause-ranking/diagnostic-planning/repair-planning）
- 同步脚本：`sync_rag_five_tool_contracts.py --base-url http://<rag-host>:8000`，TARGETS 含 5 个工具
- 联调示例：`examples/orchestrator/README.md`（每个工具 2 个 JSON block，测试取第 2 个作响应）
- 契约测试：`tests/test_five_tool_contracts.py` + `tests/test_orchestrator_examples.py`
- 历史：RAG 0.8.0→0.8.1 新增 `troubleshooting`（predictions[]）、`predictions`/`probability_source`/`probability_degraded`（repair/diagnostic 顶层），input required 改为 `['brand','dtc_codes']`（model 变可选）
