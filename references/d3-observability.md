# D3 — Observability Audit Checklist

"When it breaks at 3am, what will the on-call see?"

## Logging
- [ ] Missing logs at critical points — auth events (login/logout/fail), mutations (create/update/delete), external calls, error boundaries
- [ ] Logs at wrong level — `DEBUG` for auth failures (should be `WARN`/`ERROR`), `INFO` for stack traces (should be `ERROR`)
- [ ] Logs that print PII/tokens/passwords — `grep -rn "password\|token\|secret\|creditCard"` in log statements
- [ ] No correlation ID / trace ID — can't follow a request across service boundaries
- [ ] Structured logging missing — plain string concatenation instead of `{userId, action, result}`
- [ ] Log message doesn't tell you what happened — "Error occurred" instead of "Failed to process payment for order X: insufficient funds"

## Metrics
- [ ] No monitoring on error rates — 5xx count, business logic failures
- [ ] No latency percentiles (p50/p95/p99) — average latency hides outliers
- [ ] No saturation metrics — queue depth, connection pool utilization, thread pool size
- [ ] Missing SLI/SLO for user-facing journeys — checkout, login, search
- [ ] Alert thresholds set once and never tuned (alert fatigue) — check if anyone acknowledges pages
- [ ] No business metrics — orders placed, payments succeeded/failed, signups (not just technical metrics)

## Error Handling
- [ ] Swallowed exceptions — `except: pass`, `.catch(() => {})`, `catch {}` (Go), empty `on_error`, `except Exception as e: pass`
- [ ] Overly broad `catch Exception` / `catch (e)` that masks real bugs instead of letting them crash-loops (which is safer)
- [ ] Error messages that leak internal details — stack frames, SQL snippets, file paths, library versions
- [ ] Missing fallback for non-critical dependency failures — if recommendations service errors, page should still render core content
- [ ] Fail-open vs fail-closed混淆 — payment gateway timeout: did we assume success (double charge) or failure (lost revenue)?
- [ ] Error code/message inconsistencies — same error returns different codes across endpoints

## Completion Criterion
Any production incident can be detected from metrics, located via logs, and root-caused from error context — without reproducing locally.

## How to Report
Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D3:**
- No logging on auth/security events, silent failure on payment mutations → CRITICAL
- Swallowed exceptions on critical path, PII in logs → HIGH
- Missing SLO, unstructured logging, overly broad catch → MEDIUM
- Missing business metrics → LOW
