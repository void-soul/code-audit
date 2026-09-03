# D8 — Architecture & Code Quality Audit Checklist

Checks for dead code, god files, architecture violations — problems that "compile fine but should be refactored."

> **类型标注**：本维度以【通用】为主。架构违规判定基准 = Phase 0 用户确认的架构原则（唯一基准）。

## Dead Code 【通用】

### What is dead code
- **Unused functions/methods**: defined but never called
- **Unused variables/constants**: declared but never read
- **Branches that are always false**: `if (false) { ... }`
- **Deprecated API endpoints**: route exists but has no callers
- **Commented-out code blocks**: commented code not restored for a long time

### Detection

**grep for unused symbols:**
```bash
grep -rn "def my_function" src/
grep -rn "function my_function" src/
grep -rn "class MyClass" src/
grep -rn "my_function()" src/ | wc -l   # should be > 1: definition + usage
```

**AST static analysis (more accurate):**
```bash
flake8 --select=F401,F841 src/        # Python
tsc --noEmit                          # TypeScript
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

## Orphaned / Isolated Code（孤岛）【通用】

**是什么**：与其他部分零依赖且零被依赖的文件/模块/子项目；功能与主流程无任何关联。

**Detection**：
- 依赖图中入度=0 且出度=0 的节点（子项目级依赖图见 Phase 1 产出）
- 全库搜索该模块名，无任何引用
- 子项目未出现在解决方案/构建图中，也不被任何部署引用

**Severity**：整模块孤岛=HIGH（疑似废弃或漏接线的功能）；单文件孤岛=MEDIUM。
确认废弃=删；疑似漏接线=确认调用链是否断裂（可能是 bug）。

---

## God File (File/Class/Module) 【通用】

### What is a god file
- Excessive line count（**本次审计基准：≥ 1000 行即上帝文件**）
- Too many responsibilities (violates the Single Responsibility Principle)
- Depends on too many other modules
- Contains multiple unrelated business logic paths

### Detection

```bash
find src/ -name "*.cs" -o -name "*.cpp" -o -name "*.ts" | xargs wc -l | sort -rn | head -20
radon cc src/ -s | sort -rn | head -20        # Python
npx eslint --rule 'complexity: [error, 10]' src/
grep -c "^import\|^from" file.py              # import count
```

### Severity guide

| Metric | CRITICAL | HIGH | MEDIUM |
|--------|----------|------|--------|
| File line count | ≥ 1000 | 500–999 | 300–499 |
| Cyclomatic complexity | > 50 | 20-50 | 10-20 |
| Import count | > 30 | 15-30 | 10-15 |
| Unrelated modules | > 5 | 3-5 | 2-3 |

**God Class/Module judgment:**
- A class takes on more than 3 unrelated responsibilities
- A module imports more than 50% of the other modules
- A file contains multiple unrelated functional functions

**报告时给出**：职责拆分建议（该文件承担的 N 个职责 + 各自应拆到哪）。

---

## Architecture Violations 【通用】

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

**判定基准**：优先对照 Phase 0 用户确认的架构原则（分层规则、依赖方向、模块边界、红线）；
用户未给出时才用上述默认分层图。

**Violation patterns:**
- Presentation directly accesses Infrastructure
- Domain depends on Application
- Infrastructure depends on Presentation
- Cross-layer calls bypass the middle layer

### Detection

```bash
pydeps src/ --max-bacon=2 --cluster       # Python
npx madge --circular src/                 # Node.js
grep -rn "from.*presentation" src/domain/ # import direction check
```

### Severity guide

| Violation | Severity | Example |
|-----------|----------|---------|
| 触碰用户明示红线 | CRITICAL | Phase 0 明确禁止的依赖 |
| Domain → Application | CRITICAL | Domain model depends on an application service |
| Presentation → Infrastructure | HIGH | Controller directly accesses the database |
| Circular dependency | HIGH | A depends on B, B depends on A |
| Cross-layer call | MEDIUM | Bypasses the middle layer |
| Tightly coupled modules | MEDIUM | Too many imports between modules |

---

## Reusability & Duplication（可复用 / 提取）【通用】

### Detection

```bash
pip install similard && similard src/      # or dupfinder
```

**Manual signal checks:**
- **相同/相似代码块出现 ≥3 处** → HIGH，应提取公共模块/工具函数；报告时给出提取目标位置 + 各调用点清单
- **跨子项目复制粘贴的整段逻辑** → HIGH（还应考虑下沉为共享库）
- 两个高度相似块（> 80%）→ MEDIUM；轻度相似（60-80%）→ LOW
- The same error-handling pattern copied multiple times
- Similar validation logic scattered across the codebase

---

## Completion criterion

- No file over 500 lines (unless there's a clear reason)；≥1000 行必须报告为 CRITICAL
- No function with cyclomatic complexity > 20
- No cross-layer violations（对照 Phase 0 原则）
- No circular dependencies
- Dead code ratio < 5%
- 子项目级依赖图无未定性的孤岛节点

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D8:**
- God file (≥ 1000 lines) → CRITICAL
- Architecture violation (cross-layer call, circular dependency, 红线) → HIGH/CRITICAL
- 孤岛模块 → HIGH；Dead code (public API not called) → MEDIUM
- Duplication (> 3 places, 跨子项目复制) → HIGH/MEDIUM
- Code hygiene (unused import, small commented block) → LOW
