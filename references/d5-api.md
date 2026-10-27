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

## Multi-End Consistency（多端业务逻辑一致性）【通用｜适用条件：项目存在多端】

> 端 = 服务端 API、Web 前端、桌面端、移动端、脚本/CLI、小程序、第三方集成。
> 单端项目本节整体标 N/A。端清单与规则对照表在 Phase 1 产出（`ENDS.md`）。

**先确认端清单与职责边界**（谁负责算、谁负责展示），再逐条比对：

- [ ] **同一业务规则多端重复实现且不一致**：金额/折扣计算、权限判定、状态流转、有效期与超时、排序与分页规则、校验规则
- [ ] **状态/枚举集合不一致**：前端有后端没有的状态；后端返回前端不识别的 code（→ 落到 default 分支静默失败）
- [ ] **错误码/返回码集合不一致**：同一语义两端码值不同或一端缺失
- [ ] **字段语义/单位/时间格式不一致**：秒 vs 毫秒、UTC vs 本地时间字符串、金额单位元 vs 分、ID 类型（string vs number）精度丢失
- [ ] **本地校验与服务端校验规则不一致**：前端放行、后端拒绝（用户视角"能点却报错"）
- [ ] **客户端本地缓存/离线数据与服务端不一致策略缺失**：乐观更新失败无回滚、离线编辑无冲突解决
- [ ] **特性开关/灰度生效范围不同**：同一功能在两端表现不一致
- [ ] **向前兼容缺口**：服务端新增/改名字段，旧端未识别 → 空值/默认值处理缺失（必查 DTO 变更史）
- [ ] **版本兼容矩阵缺失**：旧客户端 × 新服务端 无声明与验证
- [ ] **错误处理路径不一致**：一端有兜底一端直接崩；一端超时 3s 一端 30s

**检测方法**：
- 对齐各端的 DTO / model / schema 定义（通常各端各写一份）→ 比对字段、默认值、可空性
- 搜索同一业务常量/阈值/状态码表在多端的重复定义（价格、折扣、超时、状态枚举）
- 取真实响应样本与各端解析代码比对（字段名、类型、可空）
- 关键流程按"端 → 步骤 → 判断条件"列表逐端走查

**报告要求**：给出 `规则 → 各端实现位置(file:line) → 差异 → 建议单一真源（以服务端为准）`。

**严重度**：规则不一致导致金额/权限/数据错误=CRITICAL；状态集合/字段语义不一致=HIGH；文案与展示差异=LOW。

---

## Completion Criterion

Every external contract is versioned, documented, and enforceable. Breaking changes require a version bump + migration path.
多端项目：端清单完整，每条关键业务规则已给出"各端实现位置 + 是否一致"的结论。

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D5:**
- Breaking API change silently deployed, batch API with no limit exposing all data → CRITICAL
- Missing backend validation, enum addition breaking strict consumer → HIGH
- Missing API version strategy, undocumented error schema → MEDIUM
- Deprecated endpoints still live | LOW
