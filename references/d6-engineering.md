# D6 — Engineering Practice Audit Checklist

Process failures, not code failures. The "how did this get merged?" layer.

> **类型标注**：本维度以【通用】为主。

## Testing 【通用】

- [ ] Only happy-path tests exist — no boundary cases (empty input, max length, negative numbers, unicode)
- [ ] No error-path testing — does it test what happens when the DB is down? The network times out?
- [ ] No concurrency testing — two simultaneous requests on the same resource produce correct state
- [ ] Tests mock the system under test — `when(repository.findById(1)).thenReturn(...)` but `findById` is THE thing you're testing
- [ ] Flaky tests are `@Ignore`d / `@Disabled` / `skip()` instead of fixed
- [ ] No contract/integration test for cross-service boundaries — pure unit tests pass, integration breaks in prod
- [ ] Mutation testing score untested — code has 100% line coverage but 40% mutation coverage (tests don't actually verify)

（重复逻辑/复制粘贴 divergent 检查已并入 D8 的 Reusability & Duplication，此处不重复。）

## Consistency 【通用】（原通用重点清单并入）

- [ ] **前后矛盾**：同一业务规则在两处实现且逻辑不一致（两处超时值、两处校验规则）→ 必有一处是 bug，HIGH
- [ ] 配置项与文档/注释描述不符
- [ ] 命名与行为矛盾：`IsXxx` 返回"是否可以"而非"是否是"；`Get` 方法有副作用
- [ ] 状态机/枚举定义与实际分支处理不覆盖一致
- [ ] **注释与代码不符**：注释说"不做 X"但代码做了 X（或反之）→ 误导性注释 HIGH
- [ ] 注释列举的参数/分支与实际签名/分支不符
- [ ] 注释提到的文件名、类名、事件名在代码中不存在（改代码没改注释）
- [ ] TODO/FIXME 旁的代码已实现该 TODO；过时注释 → MEDIUM
- [ ] **多端同一业务规则实现不一致**（前端与服务端各算一遍）→ 详见 D5 Multi-End Consistency

## Configuration 【通用】

- [ ] Environment-specific values hardcoded — IPs, domains, table names, bucket names in code instead of config/env
- [ ] Feature flags that are never cleaned up once rolled out 100% — flag exists, default=true now, dead code behind `if (flag)`
- [ ] Secrets in config files committed to VCS — `.env` in git, `application.yml` with plaintext passwords
- [ ] Config values scattered across multiple files of different formats (.env, yaml, properties) without a single source of truth
- [ ] No validation of required env vars at startup — app starts, fails on first request with cryptic error

## Build & Deploy 【在线为主，本地参照】

- [ ] No lint/format/typecheck gate in CI — broken windows get committed
- [ ] Build artifacts not reproducible — unpinned deps, no lockfile, build at different times gives different results
- [ ] Docker images built from `:latest` base — base image changes silently break builds
- [ ] Docker runs as root — `USER root`, no `USER node` / `USER 1000` directive
- [ ] No health check endpoint — orchestrator can't tell if app is alive vs just running
- [ ] No graceful shutdown — process killed mid-request, in-flight mutations lost

本地项目无 CI/Docker 时，对应检查替换为：构建脚本是否一键可复现、发布产物是否完整（资源/依赖随包复制）。

## Documentation 【通用】

- [ ] README missing setup steps or has stale commands — `npm run dev` doesn't exist anymore
- [ ] Architecture decision records (ADR) missing for non-obvious choices — "why not X?" has no answer
- [ ] No `CONTRIBUTING.md` when project is open-source — external contributors get no onboarding
- [ ] API documentation doesn't match actual endpoints — `docs.example.com` shows routes that 404
- [ ] Changelog / release notes missing — deploy team doesn't know what changed

## Completion Criterion

CI would reject the PR if any gate is missing. A new developer can build and test from the README alone. No manual deploy steps.
同一业务规则全库只有一处权威实现；注释与代码逐节抽查一致。

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D6:**
- No tests on payment/auth logic, secrets committed to VCS → CRITICAL
- No CI gate, no integration tests on cross-service boundary, 业务规则前后矛盾, 误导性注释 → HIGH
- Missing ADR on critical architecture decision, no health check, 过时注释 → MEDIUM
- Missing CONTRIBUTING.md, stale README | LOW
