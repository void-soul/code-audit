# How This Skill Forces Sequential Execution (Without Mechanical Enforcement)

A mechanical enforcement (e.g., "agent must call tool X before Y") does not
exist in this runtime. The skill instead relies on three structural patterns
that *make skipping costly enough that agents rarely do it*:

| Pattern | Mechanism | Failure mode it prevents |
|---------|-----------|--------------------------|
| Checklists ONLY in `references/` files | SKILL.md contains zero checklist items — the agent literally cannot know what to check without loading the reference | "I already know this" illusion → agent audits from memory, misses 60% of checklist items |
| Todo list with sequential completion | Finding → report → mark done → next dimension | Agent "audits" D6/D7 in 30 seconds with 0-1 findings while D1 gets 8 |
| Dimension Summary table in output | Final report requires N rows; a missing row = visible evidence of skip | User review catches the skip; the agent knows this and self-enforces |

The design goal: not "the agent cannot skip" (impossible to guarantee without
tooling), but "skipping leaves evidence the user can see, and the agent knows
this."

## What This Pattern Requires

The pattern is NOT free. It requires:

1. **Thin SKILL.md** — process control only, zero checklist content. Any checklist item
   trapped in SKILL.md is an item the agent can fake knowing.
2. **Every dimension gets its own `references/` file** — merging "to save context"
   defeats the pattern. The file-per-dimension IS the enforcement.
3. **SKILL.md MUST explicitly say "load reference file before auditing"** — without
   this instruction, agents skip the load step.
4. **Output template includes a dimension-by-dimension breakdown** — without it,
   gaps are invisible.

## Anti-Patterns Discovered in Practice

### Empty Checklist File
A skill says "see references/X for the checklist" but the file is empty (0 bytes).
The agent loads it, finds nothing, and audits from memory — the "I already know
this" illusion. This caused the agent to miss 60%+ of issues a structured
checklist would have caught.

**Rule:** A reference file described as "the audit checklist" MUST contain
actual checklist items, grep patterns, and severity guides.

### Auto-Commit by Default
The agent commits code without explicit user confirmation. This violates
user control over the commit history.

**Rule:** The agent presents the diff and proposed commit message. The user
must explicitly confirm before any `git commit` runs. No auto-commit.

### Hardcoded Project Paths
A skill hardcodes specific project paths (e.g., `attack-front/deploy`,
`attack-service/dist`) in scripts, making the skill non-portable and
inappropriate for other projects.

**Rule:** Exclusion lists and project-specific paths must be configurable
per project, not baked into the skill script.
