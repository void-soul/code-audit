# D7 — Defensive Details Audit Checklist

The small things that become big 5am pages. Numeric precision, time, magic constants, recursion, unreasonable logic, crashes.

> **类型标注**：【通用】（每次必查）/【本地】（仅定位=本地时查）。
> 定位不符的项跳过，在报告中标注 N/A。混合项目按子项目定位过滤。

## Hardcoded Values 【通用】

- [ ] Magic numbers/strings without named constants — `if status == 3` (what is 3?), `timeout = 30` (30 what? seconds? minutes?)
- [ ] IPs, ports, and domain names in code instead of config — `http://10.0.0.5:8080/api` hardcoded
- [ ] Pagination defaults hardcoded at call sites — `{ limit: 20 }` scattered everywhere instead of `DEFAULT_PAGE_SIZE`
- [ ] Retry/backoff parameters hardcoded — `sleep(5)` instead of `EXPONENTIAL_BACKOFF_INITIAL`
- [ ] Timeout values missing from external calls — `requests.get(url)` with no `timeout=` kwarg (hangs forever)

## Date & Time 【通用】

- [ ] Naive datetime compared across timezones — `datetime.now()` vs `datetime.now(tz)` when users are global
- [ ] Server-local time stored without timezone — `new Date().toISOString()` in JS but server moves to different TZ
- [ ] Floating holidays (Lunar New Year, US Thanksgiving) handled with date arithmetic that doesn't account for them
- [ ] Leap year / leap second / DST transition edge cases
- [ ] Millisecond vs microsecond vs nanosecond precision confusion — `Date.now()` (ms) vs `time.time()` (s) vs `Instant.now()` (ns)
- [ ] Time comparison with floating point — `if (timeout - elapsed > 0.001)` unreliable with float math

## Numeric 【通用】

- [ ] Money as `float`/`double` — `price = 19.99` in float, accumulate 1000 of them and cents are off
- [ ] `==` comparison on floats — `if (balance == 0.0)`, `if (totalAmount === 0.1 + 0.2)` in JS
- [ ] Integer overflow on large IDs, counters, timestamps (Year 2038 problem for 32-bit)
- [ ] Division by zero guards missing — `total / count` when count may be 0
- [ ] Floating point equality in map keys / Set membership — `Set([0.1 + 0.2])` doesn't contain `0.3`
- [ ] Rounding mode unspecified — `0.5` rounds to `0` or `1`? Banker's rounding vs arithmetic rounding

## Recursion 【通用】

- [ ] Recursive traversal of user-controlled depth without a limit — `renderTree(node.children)` where user can nest 10000 deep
- [ ] No base case visible or base case unreachable due to an off-by-one bug
- [ ] Mutual recursion (A calls B, B calls A) without a termination guarantee
- [ ] Stack overflow risk deep enough to crash the process (no tail-call optimization in most languages)

## Unreasonable Logic 【通用】（原通用重点清单并入）

- [ ] 数据集上明显的 O(n²)/O(n³)：嵌套全量遍历、循环内查库/查文件 → 导致错误结果=CRITICAL，仅性能=HIGH
- [ ] 可能死循环/无退出条件的循环 → CRITICAL
- [ ] 吞异常后继续执行的非法状态流转
- [ ] 反直觉分支：`if (!flag) return;` 后又重复检查同一 flag
- [ ] 用字符串比较做本应是类型/枚举的判断且大小写处理不一致
- [ ] 魔法数字/隐式约定无解释（尤其时间、字节、协议常量）

## Crash & Unhandled Exceptions 【本地】（原本地专用清单并入）

- [ ] `async void` / fire-and-forget `Task.Run(...)` 不 await 不观察 → 异常直接崩进程
  ```bash
  grep -rn "async void" src/ --include="*.cs"
  grep -rn "Task.Run" src/ --include="*.cs"   # 逐个核对返回 Task 是否被观察
  ```
- [ ] 事件/回调路径空引用：回调触发早于初始化完成、晚于销毁（与 D2"销毁后回调仍到达"竞态联动）
- [ ] 未捕获异常边界：线程池线程、Timer 回调、native 回调进入托管、UI 事件处理器——顶层是否有兜底 handler 与日志
- [ ] 外部数据（文件/网络/用户输入）驱动的下标/除法/转换——`Parse` vs `TryParse`
- [ ] **启动路径**：配置缺失、DLL 缺失、路径不存在、单实例互斥量异常——优雅提示 or 闪退 or 静默失败；静默失败（catch 后不日志不提示）也算缺陷

**严重度**：常见路径必现崩溃=CRITICAL；偶发竞态崩溃=HIGH；启动路径静默失败=HIGH；空引用防御缺失=HIGH/MEDIUM。

## Completion Criterion

No magic constants. All arithmetic on money uses safe numeric types. All datetime operations are timezone-explicit. Recursive depth is bounded.
本地项目：全部 `async void` 与未观察 `Task` 已逐个定性；启动路径缺失配置/文件/DLL 的行为已走查。

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D7:**
- Money arithmetic with float on financial totals, missing division-by-zero guard on critical computation, 死循环风险 → CRITICAL
- Naive timezone comparison on user-facing timestamps, float equality on payment amounts, 常见路径崩溃 → HIGH
- Magic numbers on non-critical paths, missing named constants, 启动路径静默失败 → MEDIUM/HIGH
- Unnamed pagination default | LOW
