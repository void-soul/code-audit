# D6 — Engineering Practice Audit Checklist

Process failures, not code failures. The "how did this get merged?" layer.

## Testing
- [ ] Only happy-path tests exist — no boundary cases (empty input, max length, negative numbers, unicode)
- [ ] No error-path testing — does it test what happens when the DB is down? The network times out?
- [ ] No concurrency testing — two simultaneous requests on the same resource produce correct state
- [ ] Tests mock the system under test — `when(repository.findById(1)).thenReturn(...)` but `findById` is THE thing you're testing
- [ ] Flaky tests are `@Ignore`d / `@Disabled` / `skip()` instead of fixed
- [ ] No contract/integration test for cross-service boundaries — pure unit tests pass, integration breaks in prod
- [ ] Mutation testing score untested — code has 100% line coverage but 40% mutation coverage (tests don't actually verify)
- [ ] Duplicate logic across ≥2 locations — same logic scattered should be extracted to a shared function/module
- [ ] Copy-paste blocks with divergent behavior — identical code where one copy was modified but the other wasn't

## Configuration
- [ ] Environment-specific values hardcoded — IPs, domains, table names, bucket names in code instead of config/env
- [ ] Feature flags that are never cleaned up once rolled out 100% — flag exists, default=true now, dead code behind `if (flag)`
- [ ] Secrets in config files committed to VCS — `.env` in git, `application.yml` with plaintext passwords
- [ ] Config values scattered across multiple files of different formats (.env, yaml, properties) without a single source of truth
- [ ] No validation of required env vars at startup — app starts, fails on first request with cryptic error

## Build & Deploy
- [ ] No lint/format/typecheck gate in CI — broken windows get committed
- [ ] Build artifacts not reproducible — unpinned deps, no lockfile, build at different times gives different results
- [ ] Docker images built from `:latest` base — base image changes silently break builds
- [ ] Docker runs as root — `USER root`, no `USER node` / `USER 1000` directive
- [ ] No health check endpoint — orchestrator can't tell if app is alive vs just running
- [ ] No graceful shutdown — process killed mid-request, in-flight mutations lost

## Documentation
- [ ] README missing setup steps or has stale commands — `npm run dev` doesn't exist anymore
- [ ] Architecture decision records (ADR) missing for non-obvious choices — "why not X?" has no answer
- [ ] No `CONTRIBUTING.md` when project is open-source — external contributors get no onboarding
- [ ] API documentation doesn't match actual endpoints — `docs.example.com` shows routes that 404
- [ ] Changelog / release notes missing — deploy team doesn't know what changed

## Completion Criterion
CI would reject the PR if any gate is missing. A new developer can build and test from the README alone. No manual deploy steps.

## How to Report
Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D6:**
- No tests on payment/auth logic, secrets committed to VCS → CRITICAL
- No CI gate, no integration tests on cross-service boundary → HIGH
- Missing ADR on critical architecture decision, no health check → MEDIUM
- Missing CONTRIBUTING.md, stale README | LOW
