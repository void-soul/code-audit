# D8 — Architecture & Code Quality Audit Checklist

检查死代码、上帝文件、架构违规。这些是"能编译通过但应该重构"的问题。

## 死代码（Dead Code）

### 什么是死代码
- **未使用的函数/方法**：定义了但从未被调用
- **未使用的变量/常量**：声明后从未读取
- **未使用的导入**：import了但没用到
- **永远为False的条件分支**：if (false) { ... }
- **已废弃的API端点**：路由存在但无调用方
- **注释掉的代码块**：长时间未被恢复的注释代码

### 检测方法

**grep检测未使用符号：**
```bash
# 查找函数定义但无调用
grep -rn "def my_function" src/
grep -rn "function my_function" src/
grep -rn "class MyClass" src/

# 统计引用次数（应该 > 1，定义+使用）
grep -rn "my_function()" src/ | wc -l
```

**AST静态分析（更准确）：**
```bash
# Python: pyflakes / flake8 --select=F401,F841
flake8 --select=F401,F841 src/

# TypeScript: tsc --noEmit
tsc --noEmit

# ESLint规则
npx eslint --rule 'no-unused-vars: error' src/
```

**手动检查信号：**
- 函数定义后有长时间（> 3个月）无调用
- 函数体为空或只有 `pass` / `return`
- 条件判断 `if (false)` 或 `if (never_true)`
- 被注释掉且与相邻代码重复的代码块

### 判定标准

| 情况 | Severity | 说明 |
|------|----------|------|
| 公共API函数未被调用 | HIGH | 可能是废弃接口，需确认 |
| 私有函数未被调用 | MEDIUM | 内部死代码，可清理 |
| 导入未使用 | LOW | 代码卫生问题 |
| 注释掉的代码块（>10行） | MEDIUM | 死代码残留 |
| 条件分支永远不执行 | HIGH | 逻辑错误或死代码 |

---

## 上帝文件（God File/Class/Module）

### 什么是上帝文件
- 文件行数过多（通常 > 500行）
- 承担过多职责（违反单一职责原则）
- 依赖过多其他模块
- 包含多种不相关的业务逻辑

### 检测方法

**行数检测：**
```bash
# 统计项目文件行数
find src/ -name "*.py" -exec wc -l {} + | sort -rn | head -20
find src/ -name "*.ts" -exec wc -l {} + | sort -rn | head -20
find src/ -name "*.js" -exec wc -l {} + | sort -rn | head -20
```

**复杂度检测：**
```bash
# Python: radon
radon cc src/ -s | sort -rn | head -20

# TypeScript: complexity plugin
npx eslint --rule 'complexity: [error, 10]' src/
```

**依赖分析：**
```bash
# Python: 统计import数量
grep -c "^import\|^from" file.py

# TypeScript: 统计import数量
grep -c "^import" file.ts
```

### 判定标准

| 指标 | CRITICAL | HIGH | MEDIUM |
|------|----------|------|--------|
| 文件行数 | > 1000行 | 500-1000行 | 300-500行 |
| 圈复杂度 | > 50 | 20-50 | 10-20 |
| Import数量 | > 30 | 15-30 | 10-15 |
| 功能模块数 | > 5个无关模块 | 3-5个 | 2-3个 |

**God Class/Module判定：**
- 一个类承担了3种以上不相关的职责
- 一个模块导入了项目50%以上的其他模块
- 一个文件包含多个不相关的功能函数

---

## 架构违规（Architecture Violations）

### 检查层次结构

**典型分层架构：**
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

**违规模式：**
- Presentation层直接访问Infrastructure层
- Domain层依赖Application层
- Infrastructure层依赖Presentation层
- 跨层调用绕过中间层

### 检测方法

**依赖图分析：**
```bash
# Python: 使用 pydeps
pydeps src/ --max-bacon=2 --cluster

# TypeScript: 使用 madge
npx madge --circular src/
npx madge --image graph.svg src/
```

**导入方向检查：**
```bash
# 检查是否违反依赖规则
# 例如：domain层不应import application层
grep -rn "from.*application" src/domain/
grep -rn "from.*presentation" src/domain/
grep -rn "from.*presentation" src/infrastructure/
```

**循环依赖检测：**
```bash
# Node.js
npx madge --cycles src/

# Python
python -m pyclbr -q src/  # 或使用 pydeps
```

### 判定标准

| 违规类型 | Severity | 示例 |
|---------|----------|------|
| Presentation → Infrastructure | HIGH | Controller直接访问数据库 |
| Domain → Application | CRITICAL | 领域模型依赖应用服务 |
| Infrastructure → Presentation | HIGH | 基础设施依赖UI组件 |
| 循环依赖 | HIGH | A依赖B，B依赖A |
| 跨层调用 | MEDIUM | 跳过中间层直接调用 |
| 紧耦合模块 | MEDIUM | 模块间导入过多 |

---

## 代码重复（Duplication）

### 检测方法

**文本相似度：**
```bash
# 使用 similarity检测工具
pip install similard
similard src/

# 或使用 dupfinder
dupfinder src/
```

**人工检查信号：**
- 相似函数体出现在不同文件中
- 相同的错误处理模式复制多次
- 相似的验证逻辑散落各处

### 判定标准

| 重复度 | Severity |
|--------|----------|
| 3处以上相同代码块 | HIGH |
| 2处高度相似（>80%） | MEDIUM |
| 轻微相似（60-80%） | LOW |

---

## 完成标准

- 无行数 > 500 的文件（除非有明确理由）
- 无圈复杂度 > 20 的函数
- 无跨层违规调用
- 无循环依赖
- 死代码比例 < 5%

## 如何报告

记录：`file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D8:**
- 上帝文件（>1000行）→ CRITICAL
- 架构违规（跨层调用、循环依赖）→ HIGH
- 死代码（公共API未调用）→ MEDIUM
- 代码重复（>3处）→ MEDIUM
- 代码卫生（导入未使用、小注释块）→ LOW
