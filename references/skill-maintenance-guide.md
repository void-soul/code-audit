# How to Update This Skill Without Breaking Sequential Execution

This skill uses a **thin SKILL.md + references/** pattern to force the agent
through a phased workflow (Phase 0 positioning → Phase 1 layered analysis →
Phase 2 dimension audit → Phase 3 documents → Phase 4 importance-sorted task
list) without skipping. If you edit this skill, preserve that structure.
Fix tracking / re-audit are out of scope by design.

## Architecture (WHY)

The problem: agents read a monolithic document once, produce "I know this"
illusion, and skim or skip later sections. For a multi-pass audit, D1 gets
thorough coverage while the later dimensions are gone in 30 seconds.

The solution:

```
code-audit/
├── SKILL.md                    # protocol only (phases, execution loop, output format)
└── references/
    ├── p0-scope.md             # Phase 0: 定位（在线/本地）+ 排除目录 + 架构原则（交互式）
    ├── progress-log.md         # PROGRESS.md 进度协议：开工即建、随做随记、恢复检查（防上下文丢失）
    ├── layered-analysis.md     # Phase 1: 子项目定位 → 模块定位 + 文档模板
    ├── d1-security.md          # 15-20 checklist items + grep commands + severity guide
    ├── d2-concurrency.md       # 含资源/内存泄露与 UI 阻塞【本地】节
    ├── d3-observability.md
    ├── d4-data.md              # 含 Service Performance【在线】节
    ├── d5-api.md
    ├── d6-engineering.md       # 含 Consistency 节（前后矛盾/注释不符【通用】）
    ├── d7-defensive.md         # 含 Unreasonable Logic【通用】与 Crash【本地】节
    ├── d8-architecture.md      # 含孤岛/可复用提取【通用】节；上帝文件 ≥1000 行
    ├── audit-doc-template.md
    └── skill-maintenance-guide.md
```

SKILL.md contains ZERO checklist items. The agent MUST load each reference
file to know what to check. This is enforced, not suggested.

**Type tags（类型标注）**: checklist items are tagged 【通用】/【在线】/【本地】.
【通用】 always runs; 【在线】/【本地】 items are filtered by the Phase 0 confirmed
positioning (per subproject for mixed projects). Skipped items must be
reported as N/A, never silently dropped. Do NOT split type-specific items
back into separate D9/D10-style files — the tagging IS the mechanism.

**Fact conditions（事实条件）**: sections may also carry 「适用条件：存在数据库 / 存在多端」.
When the condition is false the section is reported as N/A (not "clean"), and the
corresponding inventory doc (`SCHEMA.md` / `ENDS.md`) is skipped. Fact conditions
are confirmed in Phase 0 (Step 0.1b) and captured in the audit document header.

## Three Enforcement Mechanisms

| Mechanism | What it prevents |
|-----------|-----------------|
| Checklists ONLY in reference files | Agent can't audit without loading — breaks "I already know it" illusion |
| Todo list (D1–D8) with sequential completion | Agent must actively check off dimensions; todo state makes skips visible |
| Dimension-by-dimension summary table in output | A missing row = visible evidence of skip to the user |
| Phase 0 is interactive & mandatory | Prevents auditing with wrong positioning, wrong scope, wrong architecture baseline |

## Editing Rules

When you add new checklist items:
- [ ] Put them in the appropriate `references/*.md` file, never in SKILL.md
- [ ] Include the grep/search command to detect the issue
- [ ] Update the severity guide if the new item's severity is non-obvious
- [ ] Add a one-line pointer in SKILL.md under Execution Protocol if the pass name changes

When you add a NEW pass/dimension:
- [ ] Create `references/<id>-<slug>.md` with checklist, completion criterion, severity guide
- [ ] Add the pass to the Execution Protocol pass table in SKILL.md
- [ ] Add a row to the Dimension Summary table template in SKILL.md
- [ ] Add the pass to the Verification Checklist in SKILL.md
- [ ] Add the pass to the Dimension enum in `audit-doc-template.md`

NEVER:
- [ ] Inline a checklist item "just this once" — this is how the pattern degrades
- [ ] Merge two reference files to "save context" — the separation IS the enforcement
- [ ] Let the agent audit multiple dimensions "in one pass for efficiency" — this defeats the purpose

## Real-World Impact

Before this pattern: the agent read a 13KB monolithic SKILL.md, produced
comprehensive D1 findings (8 issues), decent D2 (5 issues), shallow D3 (2 issues),
and D4-D7 were each "audited" in 30 seconds with 0-1 findings.

After this pattern: the agent loaded each reference file D1-D7 in sequence,
produced 5-12 findings per dimension, and the dimension summary table made
coverage verifiable. Users could SEE that D4 was audited (not just claimed).

This pattern applies to any multi-dimensional skill where "the agent will be
thorough" is insufficient assurance.
