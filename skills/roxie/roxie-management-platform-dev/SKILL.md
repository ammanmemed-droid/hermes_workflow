---
name: roxie-management-platform-dev
description: "roxie_management_platform 项目开发技能：结构/命令/日志架构/踩坑."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows]
metadata:
  hermes:
    tags: [roxie, FastAPI, Logging, Python, Pipeline]
    related_skills: [codex-pipeline]
---

# roxie-management-platform 项目开发

Tool 中台（roxie-router-service）：FastAPI + Python 3.13，uv 管理依赖（`.venv`），git 仓库 D:\roxie_management_platform。

## 常用命令（⚠️ 必须清 PYTHONPATH）

```bash
cd /d/roxie_management_platform
env -u PYTHONPATH .venv/Scripts/python.exe -m pytest -q                    # 全量测试
env -u PYTHONPATH .venv/Scripts/python.exe -m pytest -q tests/test_xxx.py  # 单个文件
```
Hermes 会话会注入自己的 site-packages 到 PYTHONPATH，不清会导致 pydantic_core 等 ModuleNotFoundError。

## 项目结构

- `app/api/v1/endpoints/` — tools.py（invoke 入口）、health.py、rag_proxy.py
- `app/core/` — logging_config.py、log_context.py、registry.py、exceptions.py、responses.py（trace_id_ctx/echo_ctx）、nacos.py、discovery.py
- `app/services/rag_client.py` — RAG 调用（错误映射到 ToolExecutionError）
- `app/tools/` — 8 个 Tool 实现（dtc_grouping 等）
- `myskills/*/tool-schema.json` — Tool 契约（input/output schema）
- `docs/superpowers/specs/` — 设计文档（日志改造：2026-08-12-logging-phase1-design.md）
- `tests/` — 基线 45 项 + 临时验收测试（test_logging_phase1_temp.py，验收后按设计 §18.2 删除）

## 日志架构（第一批已完成，2026-08-12）

- `JsonFormatter`/`TextFormatter`/`ContextFilter`（注入 service/version/trace_id）+ `configure_logging(settings)`，重复调用不重复挂 handler；`get_app_logger()` 仅返回 logger
- `app/core/log_context.py`：`RequestLogContext` + `request_log_ctx` ContextVar（只存 tool_name/business_code/status/dtc_count/error_type，不存正文）
- middleware（trace_id_middleware）计时 + 输出完成摘要（tool.invoke.completed / http.request.completed / application.unhandled_error），结束清理 ContextVar
- 安全堆栈：只输出异常类型 + 文件/函数/行号，不输出异常 value；400/404 无堆栈；未知异常恰一条堆栈且在 ContextVar reset 前输出（带 trace_id）
- 字段白名单严格标量类型；`default=str` 已移除
- 环境变量：LOG_FORMAT(json/text)、LOG_LEVEL、ACCESS_LOG_ENABLED、LOG_INCLUDE_CALLER、LOG_MESSAGE_MAX_LENGTH

## 已接受的评审偏差（记录在 docs/review.md）

1. text 格式输出完整 trace_id + `[logger]`（设计示例为 8 位截断）——已注明为经评审接受的展示选择
2. 未知异常时完成摘要先于堆栈输出（Starlette ServerErrorMiddleware 在用户 middleware 之外）；堆栈在 middleware except 分支输出，带 trace_id，只一次

## Pitfalls

1. **Hermes 会话跑测试必须 `env -u PYTHONPATH`**（见上）
2. **codex 用 npm 装的 PATH 版**（0.147.0 稳定）：`~/.codex/.sandbox-bin/codex.exe`（0.147.0-alpha）缺 code-mode-host 无法执行命令；模型传 `-c 'model="gpt-5.6-sol"'`
3. **离线/无 Nacos 环境**：真实 invoke 会抛 `ToolExecutionError: roxie-supper-rag-service 无可用健康实例`（HTTP 500）——环境问题非代码 bug；测试用 stub `tool_registry.invoke` 规避
4. **schema 校验不强制 required**（实测 dtc_grouping_service `arguments={}` 能通过校验进 handler）——入参错误测试需 stub `ToolInputValidationError`
5. **测试 stub 后必须恢复** `tool_registry.invoke`（场景间会串）
6. 临时日志测试文件按设计 §18.2 在用户验收后删除，再重跑 45 项基线；删除前它是"临时资产"，不随业务提交
7. TestClient 断言日志用 LogCapture handler 挂 root（复用 tests/test_logging_phase1_temp.py 的写法）；text 格式事件匹配用 `event=xxx` 而非 JSON 语法

## 下一批方向（设计 §22，未实施）

Nacos 日志限频、discovery_ms/upstream_ms、AsyncClient 连接池、Prometheus/OTel、trace ID 安全边界（X-Trace-Id 合法性）、request/session ID HMAC
