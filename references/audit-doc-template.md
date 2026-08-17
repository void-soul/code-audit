# Audit Document Template

This template is used by `code-audit` to generate the structured fix task list.
The same document is updated by `code-audit` after fixes are committed, and by `code-re-audit` after verification.

---

# 📋 Code Audit Report — <scope>

| Field | Value |
|-------|-------|
| **Audit Date** | YYYY-MM-DD |
| **Audited By** | <agent/session identifier> |
| **Scope** | <files, branches, commit range> |
| **Commit / Branch** | `<hash or branch name>` |
| **Status** | AUDITING / REMEDIATING / RE_AUDITING / CLOSED |

*Status transitions:*
- `AUDITING` — audit in progress / complete, fixes not started
- `REMEDIATING` — fixes being applied
- `RE_AUDITING` — re-audit in progress
- `CLOSED` — all tasks resolved or accepted

---

## Executive Summary

| Severity | Total | TODO | DONE | VERIFIED | WONTFIX | REOPENED |
|----------|-------|------|------|----------|---------|----------|
| CRITICAL | 0 | 0 | 0 | 0 | 0 | 0 |
| HIGH | 0 | 0 | 0 | 0 | 0 | 0 |
| MEDIUM | 0 | 0 | 0 | 0 | 0 | 0 |
| LOW | 0 | 0 | 0 | 0 | 0 | 0 |
| **Total** | **0** | **0** | **0** | **0** | **0** | **0** |

---

## Fix Task List

<!-- Repeat this block for each finding/task -->

### AUDIT-YYYYMMDD-001: <title>

|| Field | Value |
||-------|-------|
|| **ID** | AUDIT-YYYYMMDD-001 |
|| **Dimension** | D1 – Security / D2 – Concurrency / D3 – Observability / D4 – Data / D5 – API / D6 – Engineering / D7 – Defensive / D8 – Architecture |
|| **Severity** | CRITICAL / HIGH / MEDIUM / LOW |
|| **Status** | TODO / IN_PROGRESS / DONE / VERIFIED / WONTFIX / REOPENED |
|| **Location** | `path/to/file.ext:line` |

#### Description

<What's wrong. Include: what happens, why it's a problem, concrete trigger scenario. NOT "this function is slow" — instead "this function loads the entire `orders` table to count rows via `len(list(cursor.fetchall()))` (O(n) memory + IO) where `SELECT COUNT(*)` would be O(index).">

#### Fix Plan

<How to fix it. Be specific: what code to write, what config to change, what to verify after.>

#### Fix Result (filled when status → DONE)

| Field | Value |
|-------|-------|
| **Fixed By** | <agent/session identifier> |
| **Commit / PR** | `<hash or PR link>` |
| **Fix Date** | YYYY-MM-DD |
| **What Changed** | <brief description of the fix> |
| **Files Modified** | `path/a.py`, `path/b.ts` |

#### Verification (filled when status → VERIFIED)

| Field | Value |
|-------|-------|
| **Verified By** | <agent/session identifier> |
| **Verify Date** | YYYY-MM-DD |
| **Method** | <how the fix was verified — test, manual review, grep, etc.> |
| **Evidence** | <test name, command output, commit link, etc.> |

---

### AUDIT-YYYYMMDD-002: <title>

| Field | Value |
|-------|-------|
| **ID** | AUDIT-YYYYMMDD-002 |
| **Dimension** | ... |
| **Severity** | ... |
| **Status** | ... |
| **Location** | ... |

#### Description
...

#### Fix Plan
...

#### Fix Result
...

#### Verification
...

---

<!-- Continue for each finding -->

---

## Remediation Log

<Chronological log of all changes. Updated each time a task transitions status.>

| Timestamp | Task ID | Action | Details |
|-----------|---------|--------|---------|
| YYYY-MM-DD HH:MM | AUDIT-YYYYMMDD-001 | STATUS_CHANGE | TODO → IN_PROGRESS — started fix |
| YYYY-MM-DD HH:MM | AUDIT-YYYYMMDD-001 | STATUS_CHANGE | IN_PROGRESS → DONE — commit `<hash>` — <brief description> |
| YYYY-MM-DD HH:MM | AUDIT-YYYYMMDD-001 | STATUS_CHANGE | DONE → VERIFIED — re-audit confirms fix correct |
| | | | |

---

## Re-Audit Trail

<Each re-audit cycle is recorded here.>

### Re-Audit #1 — YYYY-MM-DD

| Field | Value |
|-------|-------|
| **Audited By** | <agent/session identifier> |
| **Scope** | Tasks: AUDIT-YYYYMMDD-001, AUDIT-YYYYMMDD-002 (status=DONE) |
| **Input Document** | `docs/audit/AUDIT-YYYYMMDD-<scope>.md` (version: <commit hash>) |

**Result:**
- AUDIT-YYYYMMDD-001: VERIFIED — <evidence>
- AUDIT-YYYYMMDD-002: REOPENED — <reason: fix incomplete / regression / ...>
- New tasks introduced by fix: <IDs if any>

---

<!-- Continue for each re-audit cycle -->

---

## Final Sign-Off

| Field | Value |
|-------|-------|
| **Audit Closed** | YES / NO |
| **Closed By** | <agent/session identifier> |
| **Closed Date** | YYYY-MM-DD |
| **Remaining Tasks** | <list of WONTFIX + reason, or "none"> |
| **Acceptance Signature** | <usually the user approves> |
