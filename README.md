# Code Audit — Beyond Correctness

A comprehensive code audit framework: **8 dimensions**, processed **one at a time**. Produces a structured audit document with a fix task list. Supports completing tasks and updating the document with fix results. Designed for independent re-audit.

**Core principle:** Each dimension catches failure modes the others miss. Skip one at your peril. Batch them and you'll skip half-heartedly.

## Dimensions

| Dim | Name | Focus |
|-----|------|-------|
| D1 | Security | Injection, auth, data exposure, input validation, dependencies |
| D2 | Concurrency | Race conditions, locks, thread safety, async pitfalls |
| D3 | Observability | Logging, metrics, tracing, error reporting, alerting |
| D4 | Data & Storage | Schema, queries, caching, migration, consistency |
| D5 | API Contracts | Request/response validation, versioning, error handling |
| D6 | Engineering Practice | Testing, config, CI/CD, docs, build reproducibility |
| D7 | Defensive Details | Magic numbers, timezones, numeric precision, recursion depth |
| D8 | Architecture & Code Quality | Dead code, god files/classes, architecture violations, duplication |

## When to Use

- User asks for "audit", "全面审查", "代码审计", "review" beyond the obvious
- After a large feature lands, before a release cut, during a hardening sprint
- When `simplify-code` / `requesting-code-review` already ran, deeper pass needed
- Onboarding to an unfamiliar codebase and you need its risk profile

**Skip for:** Quick bug-fix PRs, typo fixes, user explicitly scopes narrow.

**This skill vs peers:**
- `simplify-code` = post-implementation cleanup (dead code, duplication, efficiency)
- `requesting-code-review` = pre-commit gate (security + tests + independent reviewer)
- **This skill** = deep multi-dimensional audit with fix task list + remediation tracking

---

## Execution Protocol (MANDATORY — DO NOT BATCH)

**⚠️ CRITICAL: Must scan the ENTIRE codebase, not just recent commits!**

Audit scope must be the whole codebase, not the latest commit or diff.

This skill has **8 dimensions**. They live in `references/` files — NOT in this document.

You MUST process them **sequentially**. Do not attempt to audit all 8 in a single pass.

At audit start, create a todo list with all 8 dimensions (D1–D8). Then for each:

1. **Announce** — output `### Auditing D<N>: <name>`
2. **Load** — call `skill_view(file_path='references/d<N>-<slug>.md')` to get the checklist
3. **Audit** — search the **entire codebase** against each item in that file (not just recent commits)
4. **Report** — output findings for this dimension (use the template below)
5. **Complete** — mark this dimension done in your todo list before moving to the next

**If you skip loading the reference file, you will miss checklist items. Do not write findings from memory.**

---

## Output Format

```
📋 CODE AUDIT REPORT — <scope>

CRITICAL (must fix before merge)
─────────────────────────────────
[D<N>] file:line — issue → fix
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

RECOMMENDED NEXT ACTIONS
────────────────────────
1. [highest severity item]
2. ...
```

**Never output "looks good" without the dimension-by-dimension breakdown.**

---

## Phase 2 — Generate Audit Document with Fix Task List (MANDATORY)

After all 8 dimensions are audited, you MUST generate an audit document.
This document is a structured fix task list + the bridge to re-audit.

### Step 1 — Collect findings

For each finding, assign:
- **Unique ID**: `AUDIT-<YYYYMMDD>-<3-digit-number>` (e.g., `AUDIT-20260101-001`)
- **Status**: `TODO` (default), `IN_PROGRESS`, `DONE`, `VERIFIED`, `WONTFIX`
- **Fix Task**: a concrete, actionable remediation plan

### Step 2 — Use the template

Load the template at `references/audit-doc-template.md` to write the document.

### Step 3 — Write the document

```
docs/audit/AUDIT-<YYYYMMDD>-<commit-hash-or-branch>.md
```

If `docs/audit/` does not exist, create it.

### Step 4 — Confirm path in your final response.

---

## Phase 3 — Complete Fix Tasks & Update Document (Post-Audit)

After generating the audit document, present it to the user and ask:

1. Which items should we fix now? (fix all CRITICAL + HIGH? specific IDs?)
2. Should we proceed with fixes, or is this a dry-run audit only?

**If the user asks you to fix:**
- Create a fix todo list from the audit document's task list
- Fix items one at a time or in batches by severity (CRITICAL first)
- After each fix:
  1. Show the diff to the user
  2. **Wait for the user to commit manually** (NEVER auto-commit)
  3. After the user commits, update the audit document:
     - Change task status: `TODO` → `DONE`
     - Add the commit hash or PR link
     - Add a brief description of what was changed and why
     - Update the Remediation Log

**After all fixes committed by the user**, the document becomes the input for re-audit.

---

## Audit Document Structure

See `references/audit-doc-template.md` for the full template.

The document MUST contain:
- **Scope**: what was audited (files, branches, commit range)
- **Fix Task List**: each finding as a task with ID, dimension, location, description, severity, fix plan, status
- **Remediation Log**: chronological log of status changes (timestamp, commit/PR, fix description)
- **Summary**: total findings by severity, fixed count, remaining count
- **Re-Audit Trail**: records of each re-audit cycle (for re-audit agent to append)

---

## Task Status Lifecycle

```
                           User commits
                              │
───→ TODO ──fix──→ IN_PROGRESS ──code change──→ DONE ──verify──→ VERIFIED
  │                    │                        │                   ↑
  │                    │                        └───regression──→ REOPENED
  │                    │                                            │
  │                    └────────────────────────────────────────────┘
  │                                              (re-fix cycle)
  │
  └─────────────────────────────────────────────────────────→ WONTFIX
                                                              (risk accepted)
```

| Status | Meaning |
|--------|---------|
| `TODO` | Not yet started |
| `IN_PROGRESS` | Fix being designed/implemented |
| `DONE` | Code change committed by user, awaiting re-audit |
| `VERIFIED` | Re-audit agent confirmed fix is correct |
| `REOPENED` | Re-audit found fix incomplete or incorrect |
| `WONTFIX` | Risk accepted, no fix planned |

---

## D8 — Architecture & Code Quality Details

### Dead Code Detection
- Unused functions/methods (defined but never called)
- Unused variables/constants (declared but never read)
- Unused imports
- Dead conditional branches (`if (false)`)
- Abandoned API endpoints
- Commented-out code blocks older than 3 months

**Tools:** `pyflakes`, `flake8 --select=F401,F841`, `tsc --noEmit`, ESLint `no-unused-vars`

### God File/Class Detection
| Metric | CRITICAL | HIGH | MEDIUM |
|--------|----------|------|--------|
| File lines | > 1000 | 500–1000 | 300–500 |
| Cyclomatic complexity | > 50 | 20–50 | 10–20 |
| Import count | > 30 | 15–30 | 10–15 |
| Unrelated modules | > 5 | 3–5 | 2–3 |

**Tools:** `radon cc`, ESLint `complexity`, `pydeps`, `madge`

### Architecture Violations
- Presentation → Infrastructure (bypassing Application layer)
- Domain → Application (domain model depending on use cases)
- Infrastructure → Presentation
- Circular dependencies between modules
- Tight coupling (too many cross-module imports)

**Tools:** `madge --circular`, `pydeps --cluster`, manual import direction check

### Code Duplication
| Duplication | Severity |
|-------------|----------|
| 3+ identical code blocks | HIGH |
| 2+ highly similar (>80%) | MEDIUM |
| Slightly similar (60–80%) | LOW |

**Tools:** `similard`, `dupfinder`

---

## Common Pitfalls

1. **Treating "it works" as "it's correct."** Audit the negative space — what *can* go wrong — not just the happy path.
2. **Reporting without evidence.** Every finding needs `file:line` + concrete trigger scenario.
3. **Equal-weight reporting.** A hardcoded JWT secret ≠ a missing docstring. Always sort by severity.
4. **Skipping dimensions because they're hard.** D1 Security and D2 Concurrency are the ones people most want to skip. Also the ones that cause worst production incidents.
5. **Not generating the audit document.** Without this document, fix tracking and re-audit are impossible.
6. **Updating code without updating the document.** The document is the single source of truth.
7. **Auto-committing.** Let the user decide when to commit. Update the document only after user commits.
8. **Mixing auxiliary tooling into core.** Dev-only scripts (doc updaters, scan tools, report generators) bundled inside the main binary rather than living as standalone scripts in `scripts/` — keeps the core lean and the build graph simple. Extract them out.
9. **UI surface for non-core workflows.** Features that a normal user would never trigger from the main screen don't need a tab in the UI — give them their own CLI entry point or static site.

---

## Verification Checklist

- [ ] **Audit Scope is ENTIRE CODEBASE** — not limited to recent commits or diff
- [ ] Todo list created with all 8 dimensions at audit start
- [ ] Each dimension's reference file loaded BEFORE auditing that dimension
- [ ] Every dimension has explicit findings or "no issues — searched for X, Y, Z"
- [ ] Every finding has: file:line, issue, fix suggestion, severity
- [ ] Report grouped by CRITICAL → HIGH → MEDIUM → LOW
- [ ] Dimension Summary table filled (not omitted)
- [ ] Dependencies scanned for CVEs (D1 reference)
- [ ] Money/arithmetic checked for safe numeric types (D7 reference)
- [ ] Auth checks verified on user-scoped endpoints (D1 reference)
- [ ] Audit document generated at `docs/audit/AUDIT-<date>-<scope>.md`
- [ ] Every task in the document has a unique ID and status
- [ ] User asked about fix scope before proceeding with remediation
- [ ] After user commits, document updated with commit hash + fix description
- [ ] **Separation of Concerns (Auxiliary vs Core) section checked** — confirm no dev-only tooling is bundled into main binary
- [ ] **D8 Architecture checked**: dead code, god files, architecture violations, circular dependencies
