# D6 — Engineering Practice Audit Checklist

Process failures, not code failures. The "how did this get merged?" layer.

> **类型标注**：本维度以【通用】为主。

## Testing 【通用】

- [ ] Only happy-path tests exist — no boundary cases (empty input, max length, negative numbers, unicode)
- [ ] No error-path testing — does it test what happens when the DB is down? The network times out?
- [ ] No concurrency testing — two simultaneous requests on the same resource produce correct state
- [ ] Tests mock the system under test — `when(repository.findById(1)).thenReturn(...)` but `findById` is THE thing you're testing
- [ ] Flaky tests are `@Ignore`d / `@Disabled` / `skip()` instead of fixed
- [ ] No contract/integration test for cross-service boundaries — pure unit tests pass, integration breaks in prod
- [ ] Mutation testing score untested — code has 100% line coverage but 40% mutation coverage (tests don't actually verify)

（重复逻辑/复制粘贴 divergent 检查已并入 D8 的 Reusability & Duplication，此处不重复。）

## Consistency 【通用】（原通用重点清单并入）

- [ ] **前后矛盾**：同一业务规则在两处实现且逻辑不一致（两处超时值、两处校验规则）→ 必有一处是 bug，HIGH
- [ ] 配置项与文档/注释描述不符
- [ ] 命名与行为矛盾：`IsXxx` 返回"是否可以"而非"是否是"；`Get` 方法有副作用
- [ ] 状态机/枚举定义与实际分支处理不覆盖一致
- [ ] **注释与代码不符**：注释说"不做 X"但代码做了 X（或反之）→ 误导性注释 HIGH
- [ ] 注释列举的参数/分支与实际签名/分支不符
- [ ] 注释提到的文件名、类名、事件名在代码中不存在（改代码没改注释）
- [ ] TODO/FIXME 旁的代码已实现该 TODO；过时注释 → MEDIUM
- [ ] **多端同一业务规则实现不一致**（前端与服务端各算一遍）→ 详见 D5 Multi-End Consistency

## Configuration 【通用】

- [ ] Environment-specific values hardcoded — IPs, domains, table names, bucket names in code instead of config/env
- [ ] Feature flags that are never cleaned up once rolled out 100% — flag exists, default=true now, dead code behind `if (flag)`
- [ ] Secrets in config files committed to VCS — `.env` in git, `application.yml` with plaintext passwords
- [ ] Config values scattered across multiple files of different formats (.env, yaml, properties) without a single source of truth
- [ ] No validation of required env vars at startup — app starts, fails on first request with cryptic error

## Build & Deploy 【在线为主，本地参照】

- [ ] No lint/format/typecheck gate in CI — broken windows get committed
- [ ] Build artifacts not reproducible — unpinned deps, no lockfile, build at different times gives different results
- [ ] Docker images built from `:latest` base — base image changes silently break builds
- [ ] Docker runs as root — `USER root`, no `USER node` / `USER 1000` directive
- [ ] No health check endpoint — orchestrator can't tell if app is alive vs just running
- [ ] No graceful shutdown — process killed mid-request, in-flight mutations lost

本地项目无 CI/Docker 时，对应检查替换为：构建脚本是否一键可复现、发布产物是否完整（资源/依赖随包复制）。

## Documentation 【通用】

- [ ] README missing setup steps or has stale commands — `npm run dev` doesn't exist anymore
- [ ] Architecture decision records (ADR) missing for non-obvious choices — "why not X?" has no answer
- [ ] No `CONTRIBUTING.md` when project is open-source — external contributors get no onboarding
- [ ] API documentation doesn't match actual endpoints — `docs.example.com` shows routes that 404
- [ ] Changelog / release notes missing — deploy team doesn't know what changed

## UI Copy & User-Facing Text（界面文案与用户提示）【适用条件：项目存在用户可见文本】

> **判定核心：文案是写给用户看的，不是写给开发者看的。**
> 用户可见文本 = 控件文本 / 菜单项 / 弹窗 / 通知 / 状态栏 / 引导语 / CLI 提示 / 错误对话框。
> **凡把内部实现、内部判断逻辑、内部处理流程写进用户可见文本，即为缺陷**——不论措辞多"自然"。

### A. 暴露内部实现与判断逻辑（最高优先级）

- [ ] **内部标识符泄漏**：类名、方法名、字段名、文件路径、SQL/表名、协议名、框架名、状态机内部状态名出现在用户可见文本
- [ ] **判定依据泄漏**：界面在陈述"系统如何判断"而不是"发生了什么、对你有什么影响"
  - ❌「根据分析，您的路径不在允许的存储根目录内，已跳过该事件」
  - ✅「无法播放该录像：文件不在录制目录中」
- [ ] **内部流程阶段泄漏**：把内部流水线阶段当作进度展示
  - ❌「正在执行第 2/3 步：解码器初始化」 → ✅「正在加载…」
- [ ] **实现手段/兜底策略泄漏**：「已降级使用缓存策略」「已切换为兼容模式」——实现手段不是用户概念
- [ ] **调试残留**：日志语句、TODO、内部 ID、`debug` 字样进了界面
- [ ] **错误细节直出**：异常类型 + 堆栈 + 文件行号直接呈现
  - ❌「操作失败：NullReferenceException at PlayerGridControl.cs:412」 → ✅「无法加载相机列表，请重启客户端后重试（错误码 E-102）」

### B. AI 口吻与自我陈述

- [ ] **第一人称自述**：「我理解您想要…」「我已为您…」「根据您的输入，我推断…」
- [ ] **助手人格化**：「小助手已准备就绪」「请问需要我做什么？」（问句代替可执行操作）
- [ ] **背景解释/自我表扬**：「为了帮助您更轻松地管理资源，本功能…」
- [ ] **把工具当人**：界面与用户对话协商，而不是提供操作与结果

### C. 可操作性（文案必须落到用户动作）

- [ ] **无下一步**：只说"操作失败"不给怎么办——必须包含「发生了什么 + 用户该做什么」
- [ ] **状态陈述无动作指向**：改写为可执行动作或明确后果
- [ ] **文案承诺 ≠ 实际行为**：说"保存到本地"实际写在别处；说"仅本次有效"实际永久生效；说"立即生效"实际需重启
- [ ] **成功提示与实际结果不符**：静默失败（未真正执行）却显示"已完成"/"成功" → 与 D3 静默失败联动
- [ ] **让用户操作内部概念**：要求用户"选择策略/配置节点/指定状态机状态"

### D. 冗余与风格

- [ ] 一条提示表达一个动作却超过 2 句 / 20 字
- [ ] **术语不统一**：同一概念在不同页面/不同端不同叫法 → 与 D5/D6 一致性联动
- [ ] 中英混排、大小写/标点风格不统一、同一按钮文案时有时无
- [ ] **文案未集中管理**：散落硬编码在各文件里（应入资源表/常量表，便于统一审校与批量替换）

### 检测方法

```bash
# 1) 抽取用户可见文本集合（按技术栈调整赋值点）
grep -rn "Text=\|Content=\|Header=\|Title=\|setText(\|MessageBox.Show\|Toast\|Snackbar\|setTitle(" src/

# 2) 判定性措辞 / 内部流程措辞
grep -rnE "\"[^\"]*(因为|由于|检测到|判断|校验|不满足|命中|策略|规则|阶段|流程|降级|兼容模式)[^\"]*\"" src/

# 3) AI 口吻
grep -rnE "\"[^\"]*(我理解|我已为您|助手|智能助手|请问需要)[^\"]*\"" src/

# 4) 内部标识符泄漏（示例，按项目符号表扩充）
grep -rnE "\"[^\"]*\b(IsOpened|_pendingRealOpen|storageRoot|null|Exception)\b[^\"]*\"" src/
```

**逐条人评三问**（对每条用户可见文本）：
1. 用户需要知道这句话吗？（→ 只写用户关心的结果/后果）
2. 里面有没有只对开发者有意义的信息？（→ 内部实现、判断依据、流程阶段一律移出）
3. 看完知道下一步做什么吗？（→ 补动作或明确后果）

**交叉核对**：文案描述的行为 = 代码实际行为？（写"已保存"的地方是否真的落盘）

### 报告要求（必须给改写后文案）

`位置(file:line) → 当前文案 → 问题类型(A/B/C/D) → 建议文案`——只说"文案不对"不算完成。

**严重度**：暴露内部实现/判定逻辑、文案与实际行为不符、错误细节直出=HIGH；无行动指引、AI 口吻、冗长解释=MEDIUM；术语/标点/大小写不一致=LOW。

---

## Completion Criterion

CI would reject the PR if any gate is missing. A new developer can build and test from the README alone. No manual deploy steps.
同一业务规则全库只有一处权威实现；注释与代码逐节抽查一致。
存在用户可见文本时：已产出用户可见文案清单，每条文案均通过"三问"判定，且每条问题都给出了改写建议。

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D6:**
- No tests on payment/auth logic, secrets committed to VCS → CRITICAL
- No CI gate, no integration tests on cross-service boundary, 业务规则前后矛盾, 误导性注释 → HIGH
- 暴露内部实现/判定逻辑, 文案与实际行为不符, 错误细节直出 → HIGH
- Missing ADR on critical architecture decision, no health check, 过时注释, 无行动指引/AI 口吻/冗长文案 → MEDIUM
- Missing CONTRIBUTING.md, stale README, 术语与标点不一致 | LOW
