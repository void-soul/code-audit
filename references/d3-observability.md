# D3 — Observability Audit Checklist

"When it breaks at 3am, what will the on-call see?" — 在线项目看值班告警，本地项目看"出了问题能否排查"。

> **类型标注**：【通用】（每次必查）/【在线】（仅定位=在线时查）。
> 定位不符的项跳过，在报告中标注 N/A。混合项目按子项目定位过滤。

## Logging 【通用】

- [ ] Missing logs at critical points — auth events (login/logout/fail), mutations (create/update/delete), external calls, error boundaries
- [ ] **静默 return 模式【本地，重点】**：入口校验失败时无日志直接 return（"不跳不播"类问题），排查时无从下手——校验拒绝必须记 warning 日志
- [ ] Logs at wrong level — `DEBUG` for auth failures (should be `WARN`/`ERROR`), `INFO` for stack traces (should be `ERROR`)
- [ ] Logs that print PII/tokens/passwords — `grep -rn "password\|token\|secret\|creditCard"` in log statements
- [ ] No correlation ID / trace ID — can't follow a request across service boundaries 【在线】
- [ ] Structured logging missing — plain string concatenation instead of `{userId, action, result}` 【在线】
- [ ] Log message doesn't tell you what happened — "Error occurred" instead of "Failed to process payment for order X: insufficient funds"

## Metrics 【在线】

- [ ] No monitoring on error rates — 5xx count, business logic failures
- [ ] No latency percentiles (p50/p95/p99) — average latency hides outliers
- [ ] No saturation metrics — queue depth, connection pool utilization, thread pool size
- [ ] Missing SLI/SLO for user-facing journeys — checkout, login, search
- [ ] Alert thresholds set once and never tuned (alert fatigue) — check if anyone acknowledges pages
- [ ] No business metrics — orders placed, payments succeeded/failed, signups (not just technical metrics)

本地项目 Metrics 弱化处理：本地应用重点核对"关键操作有日志/事件可追溯"，网络服务指标项标 N/A。

## Error Handling 【通用】

- [ ] Swallowed exceptions — `except: pass`, `.catch(() => {})`, `catch {}` (Go), empty `on_error`, `except Exception as e: pass`
- [ ] Overly broad `catch Exception` / `catch (e)` that masks real bugs instead of letting them crash-loops (which is safer)
- [ ] Error messages that leak internal details — stack frames, SQL snippets, file paths, library versions
- [ ] Missing fallback for non-critical dependency failures — if recommendations service errors, page should still render core content
- [ ] Fail-open vs fail-closed confusion — payment gateway timeout: did we assume success (double charge) or failure (lost revenue)?
- [ ] Error code/message inconsistencies — same error returns different codes across endpoints

## Completion Criterion

Any production incident can be detected from metrics, located via logs, and root-caused from error context — without reproducing locally.
本地项目：任何"不动作/静默失败"都能从日志定位到是哪条校验/哪个路径拒绝了。

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D3:**
- No logging on auth/security events, silent failure on payment mutations → CRITICAL
- Swallowed exceptions on critical path, PII in logs, 静默 return（无日志拒绝）→ HIGH
- Missing SLO, unstructured logging, overly broad catch → MEDIUM
- Missing business metrics → LOW
