# D8 — Architecture & Code Quality Audit Checklist

Checks for dead code, god files, and architecture violations — problems that "compile fine but should be refactored."

## Dead Code

### What is dead code
- **Unused functions/methods**: defined but never called
- **Unused variables/constants**: declared but never read
- **Unused imports**: imported but not used
- **Branches that are always false**: `if (false) { ... }`
- **Deprecated API endpoints**: route exists but has no callers
- **Commented-out code blocks**: commented code not restored for a long time

### Detection

**grep for unused symbols:**
```bash
# Find function definitions with no call sites
grep -rn "def my_function" src/
grep -rn "function my_function" src/
grep -rn "class MyClass" src/

# Count references (should be > 1: definition + usage)
grep -rn "my_function()" src/ | wc -l
```

**AST static analysis (more accurate):**
```bash
# Python: pyflakes / flake8 --select=F401,F841
flake8 --select=F401,F841 src/

# TypeScript: tsc --noEmit
tsc --noEmit

# ESLint rules
npx eslint --rule 'no-unused-vars: error' src/
```

**Manual signal checks:**
- Function with no call sites for a long time (> 3 months)
- Empty function bodies or only `pass` / `return`
- Conditions like `if (false)` or `if (never_true)`
- Commented-out blocks that duplicate adjacent code

### Severity guide

| Case | Severity | Notes |
|------|----------|-------|
| Public API function not called | HIGH | Possibly deprecated interface, needs confirmation |
| Private function not called | MEDIUM | Internal dead code, safe to remove |
| Unused import | LOW | Code hygiene issue |
| Commented-out code block (> 10 lines) | MEDIUM | Dead-code residue |
| Branch never executed | HIGH | Logic error or dead code |

---

## God File (File/Class/Module)

### What is a god file
- Excessive line count (usually > 500 lines)
- Too many responsibilities (violates the Single Responsibility Principle)
- Depends on too many other modules
- Contains multiple unrelated business logic paths

### Detection

**Line count:**
```bash
# Count lines across project files
find src/ -name "*.py" -exec wc -l {} + | sort -rn | head -20
find src/ -name "*.ts" -exec wc -l {} + | sort -rn | head -20
find src/ -name "*.js" -exec wc -l {} + | sort -rn | head -20
```

**Complexity:**
```bash
# Python: radon
radon cc src/ -s | sort -rn | head -20

# TypeScript: complexity plugin
npx eslint --rule 'complexity: [error, 10]' src/
```

**Dependency analysis:**
```bash
# Python: count imports
grep -c "^import\|^from" file.py

# TypeScript: count imports
grep -c "^import" file.ts
```

### Severity guide

| Metric | CRITICAL | HIGH | MEDIUM |
|--------|----------|------|--------|
| File line count | > 1000 | 500-1000 | 300-500 |
| Cyclomatic complexity | > 50 | 20-50 | 10-20 |
| Import count | > 30 | 15-30 | 10-15 |
| Unrelated modules | > 5 | 3-5 | 2-3 |

**God Class/Module judgment:**
- A class takes on more than 3 unrelated responsibilities
- A module imports more than 50% of the other modules
- A file contains multiple unrelated functional functions

---

## Architecture Violations

### Check layering

**Typical layered architecture:**
```
┌─────────────────┐
│   Presentation  │  (UI, Controllers, Views)
├─────────────────┤
│    Application  │  (Services, Use Cases)
├─────────────────┤
│      Domain     │  (Entities, Value Objects)
├─────────────────┤
│    Infrastructure│ (DB, External APIs, Config)
└─────────────────┘
```

**Violation patterns:**
- Presentation directly accesses Infrastructure
- Domain depends on Application
- Infrastructure depends on Presentation
- Cross-layer calls bypass the middle layer

### Detection

**Dependency graph analysis:**
```bash
# Python: pydeps
pydeps src/ --max-bacon=2 --cluster

# TypeScript: madge
npx madge --circular src/
npx madge --image graph.svg src/
```

**Import direction check:**
```bash
# Check for dependency-rule violations
# e.g., the domain layer must not import the application layer
grep -rn "from.*application" src/domain/
grep -rn "from.*presentation" src/domain/
grep -rn "from.*presentation" src/infrastructure/
```

**Circular dependency detection:**
```bash
# Node.js
npx madge --cycles src/

# Python
python -m pyclbr -q src/  # or use pydeps
```

### Severity guide

| Violation | Severity | Example |
|-----------|----------|---------|
| Presentation → Infrastructure | HIGH | Controller directly accesses the database |
| Domain → Application | CRITICAL | Domain model depends on an application service |
| Infrastructure → Presentation | HIGH | Infrastructure depends on UI components |
| Circular dependency | HIGH | A depends on B, B depends on A |
| Cross-layer call | MEDIUM | Bypasses the middle layer |
| Tightly coupled modules | MEDIUM | Too many imports between modules |

---

## Duplication

### Detection

**Text similarity:**
```bash
# Use a similarity-detection tool
pip install similard
similard src/

# Or use dupfinder
dupfinder src/
```

**Manual signal checks:**
- Similar function bodies in different files
- The same error-handling pattern copied multiple times
- Similar validation logic scattered across the codebase

### Severity guide

| Duplication | Severity |
|-------------|----------|
| Same code block in 3+ places | HIGH |
| Two highly similar blocks (> 80%) | MEDIUM |
| Mildly similar (60-80%) | LOW |

---

## Completion criterion

- No file over 500 lines (unless there's a clear reason)
- No function with cyclomatic complexity > 20
- No cross-layer violations
- No circular dependencies
- Dead code ratio < 5%

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D8:**
- God file (> 1000 lines) → CRITICAL
- Architecture violation (cross-layer call, circular dependency) → HIGH
- Dead code (public API not called) → MEDIUM
- Duplication (> 3 places) → MEDIUM
- Code hygiene (unused import, small commented block) → LOW
