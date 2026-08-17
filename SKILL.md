---
name: code-audit
description: "Use when auditing or reviewing code beyond correctness — covers security, concurrency, observability, data, API contracts, and engineering practice. Produces structured audit document with fix task list and supports completing those tasks with result tracking. Designed for independent re-audit."
version: 4.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [code-audit, code-review, security, concurrency, observability, quality, remediation, tasks]
    related_skills: [requesting-code-review, simplify-code, systematic-debugging, code-re-audit]
---

# Code Audit — Beyond Correctness

A comprehensive code audit framework: 7 dimensions, processed **one at a time**.
Produces a structured audit document with a fix task list.
Supports completing those tasks and updating the document with fix results.
Designed for independent re-audit.

**Core principle:** Each dimension catches failure modes the others miss.
Skip one at your peril. Batch them and you'll skip half-heartedly.

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

**⚠️ CRITICAL: 必须搜索整个代码库，不要只关注最近提交的代码！**

审计范围必须是整个代码库，而不是最近 commit 或 diff。用户已经明确要求全面审计，不要自行缩小范围。

This skill has **8 dimensions**. They live in `references/` files — NOT in this document.

You MUST process them **sequentially**. Do not attempt to audit all 8 in a single pass.

At audit start, create a todo list with all 8 dimensions (D1–D8). Then for each:

1. **Announce** — output `### Auditing D<N>: <name>`
2. **Load** — call `skill_view(file_path='references/d<N>-<slug>.md')` to get the checklist
3. **Audit** — search the codebase against each item in that file
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

## Common Pitfalls

1. **Treating "it works" as "it's correct."** Audit the negative space — what *can* go wrong — not just the happy path.
2. **Reporting without evidence.** Every finding needs `file:line` + concrete trigger scenario.
3. **Equal-weight reporting.** A hardcoded JWT secret ≠ a missing docstring. Always sort by severity.
4. **Skipping dimensions because they're hard.** D1 Security and D2 Concurrency are the ones people most want to skip. Also the ones that cause worst production incidents.
5. **Not generating the audit document.** Without this document, fix tracking and re-audit are impossible.
6. **Updating code without updating the document.** The document is the single source of truth.
7. **Auto-committing.** Let the user decide when to commit. Update the document only after user commits.

---

## Verification Checklist

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
