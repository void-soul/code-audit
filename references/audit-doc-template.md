# Audit Document Template

This template is used by `code-audit` to generate the structured audit document
with an importance-sorted fix task list. The audit delivers documents + task
list; fix tracking / re-audit are outside this skill's scope.

---

# 📋 Code Audit Report — <scope>

| Field | Value |
|-------|-------|
| **Audit Date** | YYYY-MM-DD |
| **Audited By** | <agent/session identifier> |
| **Scope** | <files, branches, commit range> |
| **Commit / Branch** | `<hash or branch name>` |
| **Status** | AUDITING / CLOSED |

---

## 定位与范围（Phase 0 结论）

| Field | Value |
|-------|-------|
| **项目定位** | 在线 / 本地 / 混合（逐子项目标注） |
| **标注项过滤** | 定位=在线 → 查【在线】跳【本地】；定位=本地 → 查【本地】跳【在线】 |
| **是否有数据库** | 有（类型 + schema 真源位置）/ 无 → 无则 D4 数据库三节标 N/A |
| **端清单** | 服务端 / Web / 桌面 / 移动 / CLI / 其他（≥2 端才执行 D5 多端一致性，否则标 N/A） |
| **是否有用户可见文本** | 有 / 无 → 决定 D6 UI 文案节是否执行 |
| **审计范围** | <目录/子项目清单> |
| **排除目录** | <目录 + 理由> |
| **架构原则** | <用户确认的原则，或"默认分层原则（已确认）"> |

> 分层文档（OVERVIEW.md / STRUCTURE.md / SUBPROJECT-*.md）位于本文档所在目录。

---

## Executive Summary

| Severity | Total |
|----------|-------|
| CRITICAL | 0 |
| HIGH | 0 |
| MEDIUM | 0 |
| LOW | 0 |
| **Total** | **0** |

---

## Fix Task List

<!-- 按重要性排序：CRITICAL → HIGH → MEDIUM → LOW；同级内按影响面（波及子项目/用户数/发生频率）排序 -->
<!-- Repeat this block for each finding/task -->

### AUDIT-YYYYMMDD-001: <title>

| Field | Value |
|-------|-------|
| **ID** | AUDIT-YYYYMMDD-001 |
| **Dimension** | D1 – Security / D2 – Concurrency & Resources / D3 – Observability / D4 – Data & Storage / D5 – API Contracts / D6 – Engineering / D7 – Defensive / D8 – Architecture |
| **Severity** | CRITICAL / HIGH / MEDIUM / LOW |
| **Status** | TODO |
| **Location** | `path/to/file.ext:line` |

#### Description

<What's wrong. Include: what happens, why it's a problem, concrete trigger scenario. NOT "this function is slow" — instead "this function loads the entire `orders` table to count rows via `len(list(cursor.fetchall()))` (O(n) memory + IO) where `SELECT COUNT(*)` would be O(index).">

#### Fix Plan

<How to fix it. Be specific: what code to write, what config to change, what to verify after.>

---

### AUDIT-YYYYMMDD-002: <title>

| Field | Value |
|-------|-------|
| **ID** | AUDIT-YYYYMMDD-002 |
| **Dimension** | ... |
| **Severity** | ... |
| **Status** | TODO |
| **Location** | ... |

#### Description
...

#### Fix Plan
...

---

<!-- Continue for each finding -->

---

## Summary

<Findings by severity; highest-risk themes; recommended fix order.>
