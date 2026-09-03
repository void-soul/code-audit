# D5 — API & Interface Contracts Audit Checklist

The seams where two systems meet. Most integration bugs live here.

> **类型标注**：本维度整体以【在线】为主（对外 API）。本地项目中，Contract Integrity
> 部分同样适用于内部模块间接口/插件接口/配置文件契约——此时按【通用】对待。

## Contract Integrity 【在线 / 本地内部接口通用】

- [ ] Field type drift between frontend parseInt and backend number — `parseInt(value) === value` vs `Number(value) === value`
- [ ] `null` vs `undefined` vs missing field — does the receiver distinguish all three? (e.g., `PATCH` semantics: "don't change" vs "set to null")
- [ ] Enum additions that break strict parsers on the consumer side (Rust `match`, Java `switch` with exhaustiveness check)
- [ ] Breaking changes landed without version bump — `v1/users` changed response shape without `v2`
- [ ] Optional field newly required — existing callers will 400
- [ ] Numerical precision lost at serialization boundary — JS `Number.MAX_SAFE_INTEGER` vs Go `int64`, Python `int` vs JS `number`

## Input Hardening 【在线】

- [ ] Backend trusts frontend validation exclusively — no re-validation on server (frontend can be bypassed with curl)
- [ ] Batch APIs with no `max_limit` — client can ask for 1M rows in one call, OOM the server
- [ ] WebSocket/message handlers without rate limiting — slow-loris-style resource exhaustion
- [ ] Missing timeout on outgoing HTTP calls — upstream hangs, pool drains, cascading failure
- [ ] Retry budget missing — network retry storm during upstream outage (retry amplification)

## Contract Documentation 【在线】

- [ ] OpenAPI/Protobuf specs don't match actual behavior — route exists in code but not in spec, or vice versa
- [ ] Deprecated endpoints still live — no `@Deprecated` marker, no sunset timeline, no redirect to new endpoint
- [ ] Error response schema undocumented — client can't distinguish "user not found" from "server error" programmatically
- [ ] Version strategy missing — no header, no URL path, no field discriminator for version negotiation

## Completion Criterion

Every external contract is versioned, documented, and enforceable. Breaking changes require a version bump + migration path.

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D5:**
- Breaking API change silently deployed, batch API with no limit exposing all data → CRITICAL
- Missing backend validation, enum addition breaking strict consumer → HIGH
- Missing API version strategy, undocumented error schema → MEDIUM
- Deprecated endpoints still live | LOW
