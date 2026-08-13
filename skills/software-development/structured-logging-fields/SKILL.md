---
name: structured-logging-fields
description: "Use when adding fields to whitelist-based structured logs."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [logging, structured-logs, observability, whitelist, fastapi, security]
    related_skills: [backend-project-delivery, codex-pipeline, test-driven-development]
---

# 白名单结构化日志加字段/事件

适用于这类任务："把 X 打印到日志"、"日志里加个 IP/请求参数/响应体"。典型代码库特征：日志 formatter 只序列化**白名单字段**（字段名 + 标量类型双重校验），并对落盘内容有安全约束（不记 URL/异常正文/敏感原文）。

## 执行流程（L2-Lite 档）

1. **只读侦察**：定位三处——
   - 白名单表（如 `EXTRA_FIELD_TYPES`，键 → 允许的类型元组）；
   - 日志发射点（HTTP middleware 的完成摘要/请求日志、上游客户端 `invoke()`）；
   - 限长工具与配置开关（`max_bytes`、`include_body` 类开关）。
2. **简短方案 + 一个确认点**：字段涉敏感数据（IP、请求体、上游载荷）时，给用户一个取舍确认——默认安全 vs 默认开。用户选"默认开"就按无开关直接打。
3. **实现**：白名单注册（标量类型）+ 发射点注入。
4. **测试**：caplog 按 event 属性断言、超长截断断言、失败路径**缺席**断言。
5. **验证**：针对性测试 + 全量回归 + 直接调 formatter 产出一份真实 JSON/text 样例给用户看。
6. **提交纪律**：只提交本任务文件，其余未跟踪目录（工具生成的 `.hermes/`、`openspec/` 等）不擅自带上；push 等明确指令。

## Pitfalls（本会话实证）

- **用户看的是"收到请求"那条日志**：用户问"哪个 IP 调了我的接口"，第一眼查的是 `收到工具调用请求`（request-received）行。只加在完成摘要上会得到"我怎么没有看到"。把字段加到用户关心的**每一条**请求生命周期日志（request + completed），或在交付说明里明确指出出现在哪条。
- **白名单静默丢弃**：未注册的键被 formatter 直接丢掉，不报错、日志里就是没有。先注册再接线。
- **只读 property 无法 monkeypatch**：`monkeypatch.setattr` 打在没有 setter 的 property 上会 `AttributeError: property has no setter`。改打**解析该值的方法**（如候选 URL 解析器 `_candidate_urls`），而不是 property 本身。
- **失败路径安全契约**：异常日志**不得**含 URL、异常正文、上游响应体（延续安全审查红线）。测试要断言这些字符串**缺席**，而不是断言错误文本出现——把"打印了安全错误"误写成"断言错误文本在日志里"是错的，正确断言是"只出现发送日志 + 不含敏感串"。
- **截断**：复用共享截断工具 + 配置的 max_bytes；用超大载荷（如 200KB）断言 UTF-8 安全截断。
- **JSON/text 双格式**：若两种 formatter 都遍历同一白名单，注册一次两边生效——但必须各产出一条样例核实。
- **无客户端信息时省略而非 null**：后台任务/内部触发无 client 信息时，字段应被过滤（`None` 不进日志），不要输出 `null`。
- **部署目录 ACL**：Codex 沙箱生成的目录（如 `*_svn_deploy_ready`）ACL 只授 `CodexSandboxUsers` 组写权限，普通账户 `Permission denied`。修复：管理员执行 `icacls "路径" /grant "用户:(OI)(CI)M" /T`；或让 Codex 沙箱自己写（它持有所需权限）。

## 支持文件

- `references/whitelist-logger-add-field-recipe.md` — roxie 平台具体配方（文件路径、字段、测试骨架、安全红线、验证命令）。

## 验证命令模板

```bash
PYTHONPATH= uv run pytest tests/test_<feature>_logging.py -q   # 针对性
PYTHONPATH= uv run pytest -q                                   # 全量回归
```

> `PYTHONPATH=` 前缀必须：Hermes 会话会给子进程注入 PYTHONPATH（hermes venv），不清会报 pydantic 依赖错配/collection errors。
