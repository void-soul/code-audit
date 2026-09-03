---
name: code-audit
description: "Use when auditing or reviewing code beyond correctness — positioning-aware (online vs local), layered analysis (project → subproject → module → file), with 8 core dimensions whose checklist items are tagged 【通用】/【在线】/【本地】 and filtered by the confirmed positioning (online: data security & performance; local: memory leaks, crashes, unhandled exceptions, performance). Produces overview/structure/per-subproject/per-module analysis docs plus an importance-sorted fix task list. Progress is checkpointed to PROGRESS.md so interrupted/compacted sessions resume without redoing completed work."
version: 6.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [code-audit, code-review, security, concurrency, performance, memory-leak, observability, quality, architecture, tasks]
    related_skills: [requesting-code-review, simplify-code, systematic-debugging]
---

# Code Audit — Beyond Correctness

A comprehensive, positioning-aware code audit framework: interactive scope
confirmation → layered architecture analysis → file-level deep audit (8 core
dimensions, items tagged 【通用】/【在线】/【本地】 and filtered by positioning) →
layered documents → importance-sorted fix task list.

**Core principle:** 先确认定位与范围，再逐层深入。Each dimension catches failure
modes the others miss. Skip one at your peril. Batch them and you'll skip half-heartedly.

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

## Workflow Overview (5 Phases — IN ORDER, DO NOT SKIP)

```
Phase 0  定位与范围确认     → 项目定位（在线/本地）用户确认 + 排除目录 + 架构原则   [交互式]
Phase 1  分层架构分析       → 各子项目定位 → 各模块定位                            → 结构文档
Phase 2  逐文件深入审计     → D1–D8 逐维度顺序执行；检查项标【通用】/【在线】/【本地】，按定位过滤
Phase 3  文档生成           → 综述 / 结构 / 子项目 / 模块分析文档
Phase 4  审计总文档         → 按重要性排序的整改任务清单
```

**进度保护（PROGRESS.md）**：开工即建、随做随记、先查再动——上下文被压缩或会话
中断后，从 PROGRESS.md 恢复现场，**已完成的任务绝不重复执行**。协议见 `references/progress-log.md`。

## Execution Protocol

**⚠️ CRITICAL: 必须搜索整个代码库（在确认的范围内），不要只关注最近提交的代码！**
审计范围必须是 Phase 0 确认的整个范围，而不是最近 commit 或 diff。不要自行缩小范围。

**⚠️ CRITICAL: Phase 0 是交互式的，必须让用户确认定位、排除目录、架构原则后才能继续。**

**⚠️ CRITICAL: 进度必须随做随记入 PROGRESS.md。上下文一旦丢失，未记录的已完成工作会全部重做。**

### Step 0 — 进度恢复检查（MANDATORY, FIRST ACTION）

1. **Load** `references/progress-log.md`
2. 检查 `docs/audit/` 下是否已有本次项目的 `PROGRESS.md`：
   - **不存在** → 全新审计：从 Phase 0 开始，完成定位后创建 PROGRESS.md
   - **已存在** → 读取进度：跳过全部 `DONE` 项，从「当前步骤」继续（定位/范围/原则不重新询问用户）

### Phase 0 — 定位与范围确认（交互式，MANDATORY）

1. **Load** `references/p0-scope.md`
2. 整体梳理项目 → 提出定位建议（在线 / 本地 / 混合，逐子项目标注）→ **让用户确认**
3. 整理出不需要审计的目录（第三方、构建产物、生成代码等）→ **询问用户是否需要补充**
4. **询问用户整体架构原则**（作为后续 D8 的判定基准；用户未给出时列出默认原则并请确认）
5. 产出（定位结论 + 审计范围 + 架构原则）写入审计文档头部，后续所有判断以此为准
6. **创建 `PROGRESS.md`**（按 `references/progress-log.md` 模板）：写入定位/范围/原则结论，Phase 0 标记 `DONE`；此后每完成一个最小工作单元立即更新

### Phase 1 — 分层架构分析

1. **Load** `references/layered-analysis.md`
2. **子项目定位**：逐个分析各子项目在架构中的定位（职责/角色/依赖方向/对外接口/风险初判）
3. **模块定位**：在各子项目内部划分模块，分析各模块在项目中的定位
4. 按 Phase 2 深入前先生成结构文档草稿（Phase 3 定稿）
5. **每完成一个子项目/模块定位，立即记入 `PROGRESS.md`**（含"已完成/待办"清单）

### Phase 2 — 逐文件深入审计（MANDATORY — DO NOT BATCH）

This skill has **8 audit passes**: D1–D8 core dimensions.

They live in `references/` files — NOT in this document.

You MUST process them **sequentially**. Do not attempt to audit multiple in a single pass.

At audit start (after Phase 0), create a todo list: **D1–D8**. Then for each:

1. **Announce** — output `### Auditing D<N>: <name>`
2. **Load** — read the corresponding `references/d<N>-*.md` with your file-reading tool to get the checklist
3. **Audit** — search the codebase (within confirmed scope) against each item in that file
4. **Report** — output findings for this pass (use the template below)
5. **Complete** — mark this pass done in your todo list **and update `PROGRESS.md`**（pass 状态 + 问题计数）before moving to the next

| Pass | Reference file | 重点 |
|------|----------------|------|
| D1 Security | `references/d1-security.md` | 注入/越权/密钥/敏感数据 |
| D2 Concurrency & Resources | `references/d2-concurrency.md` | 竞态/死锁/资源与内存泄露 |
| D3 Observability | `references/d3-observability.md` | 日志/指标/错误处理/静默失败 |
| D4 Data & Storage | `references/d4-data.md` | 查询/事务/缓存/服务性能 |
| D5 API Contracts | `references/d5-api.md` | 契约一致性/输入加固 |
| D6 Engineering | `references/d6-engineering.md` | 测试/配置/一致性（前后矛盾/注释不符） |
| D7 Defensive | `references/d7-defensive.md` | 魔法值/时间/数值/不合理逻辑/崩溃 |
| D8 Architecture | `references/d8-architecture.md` | 死代码/孤岛/上帝文件/架构违规/重复 |

### 类型标注过滤规则（每维度通用）

每个检查项/小节标注三种类型之一：

| 标注 | 含义 | 执行规则 |
|------|------|----------|
| 【通用】 | 不限定位 | **每次必查** |
| 【在线】 | 在线项目专用（数据安全、服务性能等） | 定位=在线时查；定位=本地时**跳过并在报告标 N/A** |
| 【本地】 | 本地项目专用（内存泄露、闪退、UI 阻塞等） | 定位=本地时查；定位=在线时**跳过并在报告标 N/A** |

- 混合项目：按各子项目的定位过滤（Phase 0 逐子项目标注的定位）。
- 与当前定位一致的标注项视为**重点审查项**，发现问题时优先给高严重度。

**If you skip loading the reference file, you will miss checklist items. Do not write findings from memory.**

### Phase 3 — 文档生成

按 `references/layered-analysis.md` 中的模板生成（写入 `docs/audit/AUDIT-<YYYYMMDD>-<scope>/`）：

- `OVERVIEW.md` — 项目综述文档（定位、技术栈、子项目清单、架构、关键数据流）
- `STRUCTURE.md` — 结构文档（目录树、审计范围/排除标注、模块清单）
- `SUBPROJECT-<name>.md` — 各子项目分析文档（含模块定位，可内嵌模块分析）
- 深层模块内容多时另出 `MODULE-<subproject>-<module>.md`

**每生成一份文档，立即在 `PROGRESS.md` 勾选**（防止上下文丢失后重复生成）。

### Phase 4 — 生成审计总文档（按重要性排序，MANDATORY）

After all audit passes, generate the audit document with a fix task list
**sorted by importance (CRITICAL → HIGH → MEDIUM → LOW，同级内按影响面排序)**.

1. **Collect findings** — each finding gets:
   - **Unique ID**: `AUDIT-<YYYYMMDD>-<3-digit-number>` (e.g., `AUDIT-20260101-001`)
   - **Status**: `TODO`（审计产出即交付，不做修复跟踪；执行方可自行改状态）
   - **Fix Task**: a concrete, actionable remediation plan
2. **Use the template** at `references/audit-doc-template.md`
3. **Write** to `docs/audit/AUDIT-<YYYYMMDD>-<scope>/AUDIT-<YYYYMMDD>-<scope>.md`
4. **Confirm path** in your final response.
5. 更新 `PROGRESS.md`：全部阶段标记 `DONE`，当前步骤 = `CLOSED`

---

## Output Format (per audit pass, Phase 2)

```
📋 CODE AUDIT REPORT — <scope>

CRITICAL (must fix before merge)
─────────────────────────────────
[<ID>] file:line — issue → fix
...

HIGH (fix this sprint)
─────────────────────
...

MEDIUM (backlog, but tracked)
─────────────────────────────
...

LOW (nice-to-have)
──────────────────
...

DIMENSION SUMMARY
─────────────────
D1 Security:        {N} issues | {brief note or "clean"}
D2 Concurrency:     {N} issues | {brief note or "clean"}
D3 Observability:   {N} issues | {brief note or "clean"}
D4 Data & Storage:  {N} issues | {brief note or "clean"}
D5 API Contracts:   {N} issues | {brief note or "clean"}
D6 Engineering:     {N} issues | {brief note or "clean"}
D7 Defensive:       {N} issues | {brief note or "clean"}
D8 Architecture:    {N} issues | {brief note or "clean"}
（定位不符的标注项已跳过并计 N/A：在线项 {N} 条 / 本地项 {N} 条）

RECOMMENDED NEXT ACTIONS (按重要性)
────────────────────────
1. [highest severity item]
2. ...
```

**Never output "looks good" without the dimension-by-dimension breakdown.**

---

## Audit Document Structure

See `references/audit-doc-template.md` for the full template.

The document MUST contain:
- **定位与范围**: 项目定位（在线/本地/混合，逐子项目）、审计范围、排除目录、架构原则（Phase 0 结论）
- **Fix Task List**: each finding as a task with ID, dimension, location, description, severity, fix plan — **按重要性排序**
- **Summary**: total findings by severity

---

## Common Pitfalls

1. **跳过 Phase 0 就开始审计。** 定位决定【在线】/【本地】标注项查不查，排除目录决定审计边界，架构原则决定 D8 判定基准——都没确认就审，方向全错。
2. **Treating "it works" as "it's correct."** Audit the negative space — what *can* go wrong — not just the happy path.
3. **Reporting without evidence.** Every finding needs `file:line` + concrete trigger scenario.
4. **Equal-weight reporting.** A hardcoded JWT secret ≠ a missing docstring. Always sort by severity.
5. **Skipping dimensions because they're hard.** D1/D2/D7 are the ones people most want to skip. Also the ones that cause worst incidents.
6. **无视类型标注。** 定位为本地却去报 CORS/越权、定位为在线却跳过内存泄露——【在线】/【本地】标注项必须按 Phase 0 定位过滤，不符的明确标 N/A 而非默默漏掉。
7. **Not generating the layered documents.** 没有综述/结构/子项目文档，整改清单就失去上下文。
8. **Mixing auxiliary tooling into core.** Dev-only scripts bundled inside the main binary rather than as standalone scripts in `scripts/` — extract them out.
9. **最后统一补记进度。** 每完成一个子项目/pass/文档就必须立即更新 `PROGRESS.md`；上下文一旦被压缩，未记录的已完成工作会全部重做。恢复现场时 `DONE` 项一律跳过。

---

## Verification Checklist

- [ ] **Phase 0 完成且用户已确认**：项目定位（在线/本地/混合）、排除目录、架构原则
- [ ] **Audit Scope = Phase 0 确认的范围** — not limited to recent commits or diff
- [ ] Todo list created with D1–D8 at audit start
- [ ] Each pass's reference file loaded BEFORE auditing that pass
- [ ] Every pass has explicit findings or "no issues — searched for X, Y, Z"
- [ ] Every finding has: file:line, issue, fix suggestion, severity
- [ ] Report grouped by CRITICAL → HIGH → MEDIUM → LOW
- [ ] Dimension Summary table filled（定位不符的标注项计 N/A，未默默漏掉）
- [ ] **分层文档已生成**：OVERVIEW / STRUCTURE / SUBPROJECT-*
- [ ] Dependencies scanned for CVEs (D1 reference)
- [ ] Money/arithmetic checked for safe numeric types (D7 reference)
- [ ] 定位=在线：【在线】标注项已查（数据安全/越权/注入/密钥、N+1/无分页/无超时）
- [ ] 定位=本地：【本地】标注项已查（内存泄露/事件退订/Dispose、闪退/未处理异常、UI 阻塞）
- [ ] Audit document generated at `docs/audit/AUDIT-<date>-<scope>/`，任务按重要性排序
- [ ] Every task in the document has a unique ID and status
- [ ] **`PROGRESS.md` 已在开工时创建，且随做随记**（非事后补记）
- [ ] **会话开始时执行了进度恢复检查，`DONE` 项未重做**
