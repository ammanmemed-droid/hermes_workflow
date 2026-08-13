# roxie RAG 0.8.1 契约对齐实操记录（2026-08-13）

## 环境

- RAG 服务：`http://172.16.67.169:8000`（版本探测：`GET /openapi.json` 的 `info.version`）
- 中台 sync 脚本：`D:\roxie_management_platform\sync_rag_five_tool_contracts.py`
- 中台契约：`myskills/<tool>-skill/tool-schema.json`（5 个工具）

## Sync 流程

```bash
# 1) 改脚本 expected_version 为 RAG 实际版本（0.8.0 → 0.8.1）
# 2) 跑 sync（会重拉全部 5 个工具的 input/output schema）：
cd /d/roxie_management_platform
PYTHONPATH= uv run python sync_rag_five_tool_contracts.py --base-url http://172.16.67.169:8000
# 3) 同步后跑全量测试：
PYTHONPATH= uv run pytest -q
```

## 踩坑：sync 后契约测试失败

`tests/test_orchestrator_examples.py` 从 `examples/orchestrator/README.md` 按 ```json block 顺序读取
响应示例（`index*2+1` 是响应），校验是否符合 output_schema。RAG 新增 required 字段后，
README 示例必须同步补字段，否则：

```
FAILED test_orchestrator_examples.py::test_documented_rag_080_response_satisfies_output_contract[...]
E   assert "(root): 'predictions' is a required property; ..." is None
```

修法：在 README 对应工具的响应示例 JSON 里补上新 required 字段（用真实响应数据），再跑测试。

## 0.8.1 字段变化记录

1. **入参 required**：5 个工具统一 `['brand', 'dtc_codes']`（model/year/language 可选，默认 `""`）
2. **cause_ranking**：`CausePredictionOut` 新增 `troubleshooting`（string，可选；sync 后为 required）
3. **repair_planning / diagnostic_planning**：顶层新增 `predictions[]`（CausePredictionOut 数组）、
   `probability_source`、`probability_degraded`（均 required）——即这两个接口也输出原因概率排序
4. **repair_planning**：`MvpGroupOut.shared_parts` 改名 `parts`（仅此工具；其他 4 个工具仍是
   `shared_parts`；`group_reason` 枚举值 `shared_parts`/`same_system_shared_parts` 与
   `grouping_basis` 文本是枚举/展示字符串，不是字段名，不能改）

## 中台校验机制要点

- 出参校验：`registry.py` invoke() → `strip_invalid_nulls()` + `validate_against_schema()`
- `additionalProperties: false` → RAG 返回契约外字段会抛 `ToolOutputValidationError`（code=50001）
- 区分两类情况：
  - **RAG 合法新增**（troubleshooting/predictions）→ 中台同步 schema
  - **RAG 契约外多余**（0.8.0 的 `labor_hours_source`/`reference_price_source`）→ 上游 bug，
    中台不改，等 RAG 修（0.8.1 已移除）
