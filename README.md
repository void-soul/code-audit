# Code Audit — Beyond Correctness

A positioning-aware code audit framework: interactive scope confirmation →
layered architecture analysis → file-level deep audit (**8 core dimensions,
items tagged 【通用】/【在线】/【本地】 and filtered by positioning**, processed one
dimension at a time) → layered analysis documents → importance-sorted fix task list.

**Core principle:** 先确认定位与范围，再逐层深入。Each dimension catches failure
modes the others miss. Skip one at your peril. Batch them and you'll skip half-heartedly.

## Workflow

```
Phase 0  定位与范围确认     → 项目定位（在线/本地）用户确认 + 排除目录 + 架构原则   [交互式]
Phase 1  分层架构分析       → 各子项目定位 → 各模块定位                            → 结构文档
Phase 2  逐文件深入审计     → D1–D8 逐维度顺序执行；检查项标【通用】/【在线】/【本地】，按定位过滤
Phase 3  文档生成           → 综述 / 结构 / 子项目 / 模块分析文档
Phase 4  审计总文档         → 按重要性排序的整改任务清单
```

**进度保护**：全程记录到 `docs/audit/<dir>/PROGRESS.md`（开工即建、随做随记、先查再动）。
上下文被压缩或会话中断后从 PROGRESS.md 恢复现场，已完成的任务不重复执行。
协议见 `references/progress-log.md`。

## Audit Passes

| Pass | Name | Focus |
|------|------|-------|
| D1 | Security | Injection, auth/越权, secrets, data exposure, input validation, dependencies |
| D2 | Concurrency & Resources | Race conditions, locks, resource/memory leaks【本地】, UI 阻塞【本地】 |
| D3 | Observability | Logging, metrics, error handling, 静默失败 |
| D4 | Data & Storage | Queries, transactions, caching, Service Performance【在线】, **数据库设计合理性 / 僵尸表僵尸字段 / 代码-Schema 漂移**（有数据库时） |
| D5 | API Contracts | Contract integrity, input hardening, **多端业务逻辑一致性**（多端时） |
| D6 | Engineering Practice | Testing, config, consistency（前后矛盾/注释不符）, docs |
| D7 | Defensive Details | Magic numbers, timezones, numeric precision, 不合理逻辑, 崩溃/未处理异常【本地】 |
| D8 | Architecture & Code Quality | Dead code, 孤岛, god files (≥1000 lines), architecture violations, 可复用/提取 |

每个检查项标注【通用】/【在线】/【本地】：【通用】必查；【在线】/【本地】项按 Phase 0
确认的定位过滤（混合项目逐子项目），被跳过的项在报告中明确标 N/A。
另有第三类过滤——**事实条件**：标注「适用条件：存在数据库 / 存在多端」的小节，
条件不成立时整体标 N/A（D4 的数据库三节、D5 的多端一致性节）。
原通用重点（孤岛/可复用/矛盾/上帝文件等）已并入 D6/D7/D8，
在线数据安全与性能并入 D1/D4，本地内存泄露与崩溃并入 D2/D3/D7。

## When to Use

- User asks for "audit", "全面审查", "代码审计", "review" beyond the obvious
- After a large feature lands, before a release cut, during a hardening sprint
- Onboarding to an unfamiliar codebase and you need its risk profile

**Skip for:** Quick bug-fix PRs, typo fixes, user explicitly scopes narrow.

**This skill vs peers:**
- `simplify-code` = post-implementation cleanup (dead code, duplication, efficiency)
- `requesting-code-review` = pre-commit gate (security + tests + independent reviewer)
- **This skill** = deep multi-dimensional audit with layered docs + importance-sorted fix task list

---

## Documents Produced

Written to `docs/audit/AUDIT-<YYYYMMDD>-<scope>/`:

- `OVERVIEW.md` — 项目综述文档（定位、技术栈、子项目清单、架构、关键数据流）
- `STRUCTURE.md` — 结构文档（目录树、审计范围/排除标注、模块清单）
- `SUBPROJECT-<name>.md` — 各子项目分析文档（含模块定位）
- `SCHEMA.md` — 表/列清单 + 双向差集结论（僵尸表/字段 vs 代码-Schema 漂移），仅有数据库时
- `ENDS.md` — 端矩阵 + 关键业务规则各端一致性对照，仅多端时
- `AUDIT-<YYYYMMDD>-<scope>.md` — 审计总文档：定位与范围（Phase 0 结论）+
  按重要性排序的修复任务清单（CRITICAL → HIGH → MEDIUM → LOW，同级按影响面）

Each task has a unique ID (`AUDIT-<YYYYMMDD>-<NNN>`), dimension, severity,
location, description, and a concrete fix plan. Fix tracking / re-audit are
out of this skill's scope — the task list is the deliverable.

---

## Common Pitfalls

1. **跳过 Phase 0 就开始审计。** 定位决定【在线】/【本地】标注项查不查，排除目录决定审计边界，架构原则决定 D8 判定基准。
2. **Treating "it works" as "it's correct."** Audit the negative space — what *can* go wrong — not just the happy path.
3. **Reporting without evidence.** Every finding needs `file:line` + concrete trigger scenario.
4. **Equal-weight reporting.** A hardcoded JWT secret ≠ a missing docstring. Always sort by severity.
5. **Skipping dimensions because they're hard.** D1/D2/D7 are the ones people most want to skip. Also the ones that cause worst incidents.
6. **无视类型标注。** 定位为本地却去报 CORS/越权、定位为在线却跳过内存泄露——【在线】/【本地】标注项必须按定位过滤，不符的明确标 N/A。
7. **Not generating the layered documents.** 没有综述/结构/子项目文档，整改清单就失去上下文。
8. **只读代码不读 schema。** 查"代码用了不存在的表/字段"必须先有 schema 真源清单（迁移/DDL/实体），否则只能靠印象猜。
9. **多端只看一端。** 多端一致性要求逐端比对同一规则（前端/服务端各算一遍是典型漂移源）。
10. **把不适用的条件节当成"已查无问题"。** 无数据库/单端项目，条件节标 N/A 而非 clean。
11. **Mixing auxiliary tooling into core.** Dev-only scripts bundled inside the main binary rather than as standalone scripts in `scripts/` — extract them out.

---

## Verification Checklist

- [ ] **Phase 0 完成且用户已确认**：项目定位（在线/本地/混合）、**有无数据库 + 端清单**、排除目录、架构原则
- [ ] **Audit Scope = Phase 0 确认的范围** — not limited to recent commits or diff
- [ ] Todo list created with D1–D8 at audit start
- [ ] Each pass's reference file loaded BEFORE auditing that pass
- [ ] Every pass has explicit findings or "no issues — searched for X, Y, Z"
- [ ] Every finding has: file:line, issue, fix suggestion, severity
- [ ] Report grouped by CRITICAL → HIGH → MEDIUM → LOW
- [ ] Dimension Summary table filled（定位不符的标注项计 N/A）
- [ ] **分层文档已生成**：OVERVIEW / STRUCTURE / SUBPROJECT-*
- [ ] 定位=在线：【在线】标注项已查（数据安全/越权/注入/密钥、N+1/无分页/无超时）
- [ ] 定位=本地：【本地】标注项已查（内存泄露/事件退订/Dispose、闪退/未处理异常、UI 阻塞）
- [ ] **有数据库时**：`SCHEMA.md` 已产出，D4 三节已执行（设计合理性、僵尸表/僵尸字段、代码-Schema 漂移）
- [ ] **多端时**：`ENDS.md` 已产出，D5 多端一致性已执行（每条规则有各端实现位置与是否一致结论）
- [ ] **不适用项标 N/A**（无数据库/单端），未当成 clean
- [ ] Audit document generated，任务按重要性排序，每项有唯一 ID 与 status
