# How to Update This Skill Without Breaking Sequential Execution

This skill uses a **thin SKILL.md + references/** pattern to force the agent
through 7 audit dimensions without skipping. If you edit this skill, preserve
that structure.

## Architecture (WHY)

The problem: agents read a monolithic document once, produce "I know this"
illusion, and skim or skip later sections. For a 7-dimension audit, D1 gets
thorough coverage while D6/D7 are gone in 30 seconds.

The solution:

```
code-audit/
├── SKILL.md                    # 5KB — protocol only (overview, when to use, execution loop, output format)
└── references/
    ├── d1-security.md          # 15-20 checklist items + grep commands + severity guide
    ├── d2-concurrency.md
    ├── d3-observability.md
    ├── d4-data.md
    ├── d5-api.md
    ├── d6-engineering.md
    └── d7-defensive.md
```

SKILL.md contains ZERO checklist items. The agent MUST load each reference
file to know what to check. This is enforced, not suggested.

## Three Enforcement Mechanisms

| Mechanism | What it prevents |
|-----------|-----------------|
| Checklists ONLY in reference files | Agent can't audit without loading — breaks "I already know it" illusion |
| Todo list (7 items) with sequential completion | Agent must actively check off dimensions; todo state makes skips visible |
| Dimension-by-dimension summary table in output | A missing row = visible evidence of skip to the user |

## Editing Rules

When you add new checklist items:
- [ ] Put them in the appropriate `references/d<N>-<slug>.md` file, never in SKILL.md
- [ ] Include the grep/search command to detect the issue
- [ ] Update the severity guide if the new item's severity is non-obvious
- [ ] Add a one-line pointer in SKILL.md under Execution Protocol if the dimension name changes

When you add a NEW dimension:
- [ ] Create `references/d<N>-<slug>.md` with checklist, completion criterion, severity guide
- [ ] Add the dimension to the Execution Protocol step 1 (todo list) in SKILL.md
- [ ] Add a row to the Dimension Summary table template in SKILL.md
- [ ] Add the dimension to the Verification Checklist in SKILL.md

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
