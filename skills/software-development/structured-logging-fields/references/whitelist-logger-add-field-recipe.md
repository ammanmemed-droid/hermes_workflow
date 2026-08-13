# roxie 平台：白名单日志加字段配方（2026-08-12 实证）

项目：`D:/roxie_management_platform`（FastAPI + uvicorn，日志 JSON/text 双格式）。
两轮 L2-Lite 交付：`client_ip`（完成摘要 + invoke.request）、RAG 请求/响应体。

## 架构落点

| 组件 | 位置 | 作用 |
|---|---|---|
| 字段白名单 | `app/core/logging_config.py` → `EXTRA_FIELD_TYPES` | 键→类型元组；`EXTRA_FIELDS` 固定顺序 |
| 双 formatter | 同文件 `JsonFormatter` / `TextFormatter` | 都遍历 `_extra_fields()` → 注册一次两边生效 |
| 限长 | `_truncate_utf8()` + `settings.log_request_body_max_bytes`（默认 65536） | 日志限长，不影响业务 |
| 清洗 | `_sanitize_text()`（控制字符 + FIELD_MAX_LENGTH=256） | 防日志注入 |
| HTTP 发射点 | `app/main.py`：`_log_request_completed`、`_log_invoke_request`、`_log_unhandled_error` | 请求生命周期日志 |
| 上游发射点 | `app/services/rag_client.py` → `RagServiceClient.invoke()` | RAG 请求/响应 |

## 加字段三步（实例：client_ip）

1. 白名单注册（`logging_config.py`）：
   ```python
   "client_ip": (str,),
   ```
2. 发射点注入（`main.py`）：
   ```python
   def _client_ip(request: Request) -> str | None:
       client = getattr(request, "client", None)
       return client.host if client is not None else None
   # fields 字典里： "client_ip": _client_ip(request),
   # None 会被 {k:v for ... if v is not None} 过滤 → 省略而非 null
   ```
3. 测试（`tests/test_client_ip_logging.py`）：caplog 按 `getattr(r, "event")` 过滤；断言 `record.client_ip == "testclient"`（TestClient 的 TCP 对端名）。

## 加请求/响应体（实例：rag.request / rag.response）

- `invoke()` 里 `self._client.post(...)` **之前**记 `rag.request`（含 `rag_request_body`），`resp.json()` **之后**记 `rag.response`（含 `rag_response_body`）。
- 序列化辅助（复用，不裸 dump）：
  ```python
  @staticmethod
  def _serialize_log_body(value: object, max_bytes: int) -> str | None:
      try:
          serialized = json.dumps(value, ensure_ascii=False, separators=(",", ":"))
      except (TypeError, ValueError):
          return None
      body, _truncated = _truncate_utf8(serialized, max(1, int(max_bytes)))
      return body
  ```
- 测试（`tests/test_rag_client_logging.py`）：
  - 用 `_FakeClient`（记录 `posted`）+ `_FakeResponse`（固定 payload）替换 `client._client`；
  - **monkeypatch `RagServiceClient._candidate_urls`**（类方法，返回固定 URL 列表）——不要打 `rag_discovery.direct_url`，它是只读 property（`AttributeError: property ... has no setter`）。

## 安全红线（W3 审查延续，测试必须显式断言）

- 失败路径**不得**打印：URL、异常正文、上游响应体。
- 正确断言模式：`events == {"rag.request"}` + `"connection refused" not in joined` + `"fake-rag" not in joined`。
- 错误模式（本会话踩过）：断言异常文本**在**日志里——异常根本没进日志，断言必失败。

## 部署包同步（Codex 沙箱 ACL 坑）

- 目标目录 `D:/roxie_management_platform_svn_deploy_ready` 由 Codex 沙箱生成，ACL 只授 `WANGSHIFENG\CodexSandboxUsers` 组 Modify；普通账户（`BUILTIN\Users` 仅只读）`cp` 报 `Permission denied`。
- 诊断：`powershell Get-Acl ... | Select Access`；`whoami /groups` 查组成员。
- 修复：管理员 `icacls "D:\...\roxie_management_platform_svn_deploy_ready" /grant "shifeng.wang:(OI)(CI)M" /T`；或直接让 Codex 沙箱执行复制（它持有所需权限）。
- `.env.example` 是**有意的环境差异**（部署包 `LOG_INCLUDE_REQUEST_BODY=true`，本地 false）——同步部署包时不要覆盖。

## 验证命令（PYTHONPATH 必须清）

```bash
PYTHONPATH= uv run pytest tests/test_client_ip_logging.py -q
PYTHONPATH= uv run pytest tests/test_rag_client_logging.py -q
PYTHONPATH= uv run pytest -q    # 全量：83 passed（含这两轮）
```
