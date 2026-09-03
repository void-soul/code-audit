# D2 — Concurrency & Resources Audit Checklist

Bugs that only manifest under load, and resources that never come back.

> **类型标注**：【通用】（每次必查）/【本地】（仅定位=本地时查）。
> 定位不符的项跳过，在报告中标注 N/A。混合项目按子项目定位过滤。

## Race Conditions 【通用】

- [ ] Read-modify-write without atomicity — `x += 1`, `counter++`, `collections.Counter[x] += 1` when `x` is shared across threads/async
- [ ] TOCTOU: checking a state, then acting on it — `if file.exists(): file.delete()`, `if os.path.exists(x): shutil.rmtree(x)`
- [ ] Check-then-act without lock — `if not in_dict[key]: in_dict[key] = compute()`
- [ ] Missing distributed lock for shared-resource mutations across instances (e.g., two app servers both running the same cron job)
- [ ] File write without file lock — multiple workers writing the same log/cache file
- [ ] 销毁/停止后回调仍到达的竞态：对象已 Dispose 后回调仍进入（"停止路径早退不当"类问题）【本地】

## Deadlocks 【通用】

- [ ] Lock ordering inconsistency — Thread A holds L1 → wants L2; Thread B holds L2 → wants L1
- [ ] Nested `synchronized`/`Lock`/`mutex`/`acquire()` without ordering discipline
- [ ] Holding a lock across an I/O or network call (DB query, HTTP fetch)
- [ ] `asyncio.Lock`/`threading.Lock` acquired but not released (missing `try/finally` or `async with`)
- [ ] 粗粒度大锁跨 IO 持有、自旋等待、双锁无序导致偶发阻塞 【本地】

## Resource Leaks 【通用】

- [ ] DB connections not returned to pool — no `try/finally`, no `with` context manager around `connection.cursor()`
- [ ] File handles left open — `open()` without `with` or `.close()` in finally
- [ ] Sockets / HTTP clients not closed — `aiohttp.ClientSession`, `urllib`, `requests.Session` lifecycle
- [ ] Event listeners attached without removal — `addEventListener` with no `removeEventListener`, React `useEffect` without cleanup, Vue `mounted` without `beforeDestroy`
- [ ] `setTimeout`/`setInterval` not cleared on component unmount
- [ ] Worker threads / goroutines spawned without shutdown mechanism (goroutine leak)
- [ ] Generators / iterators holding large resources open (`yield` blocks not closed)

## Memory Leaks — Managed & Native 【本地】

- [ ] C# 事件 `+=` 无对应 `-=`；Dispose/Close 路径漏掉退订 → 发布者持有订阅者无法回收
  ```bash
  grep -rn "\+=" src/ --include="*.cs" | grep -v "=="
  grep -rn "\-=" src/ --include="*.cs"
  ```
- [ ] IDisposable 创建但无 `using`/`Dispose`/`finally` 释放；异常路径提前 return 跳过释放
  ```bash
  grep -rn "new \(MemoryStream\|FileStream\|Bitmap\|Process\)" src/ --include="*.cs"
  ```
- [ ] Dispose 模式不完整：三件事缺一 —— 退订事件、停 Timer、释放 native 句柄；`GC.SuppressFinalize` 缺失
- [ ] 静态集合只增不减：static Dictionary/List 做缓存无淘汰策略
- [ ] Timer/匿名 lambda 捕获大对象；长生命周期对象注册短生命周期对象的回调
- [ ] WPF：未解除的命令绑定、大图未及时释放（BitmapImage 缓存）
- [ ] C++：new/delete 配对（含异常路径，应改 smart pointer）、malloc/free 配对、缓冲区越界、句柄错误路径未 Close
- [ ] interop：封送缓冲区释放归属不明、`Marshal.PtrToString*` 对应的非托管内存未释放

**严重度**：高频路径（每帧/每次播放/每次录制）泄露=CRITICAL；低频路径=HIGH；静态缓存无上限=MEDIUM（视增长率）。

## UI Thread Blocking 【本地】

- [ ] `.Result`/`.Wait()` 同步等待异步 —— 死锁风险，`grep -rn "\.Result\b\|\.Wait()" src/ --include="*.cs"`
- [ ] UI 线程内做同步 IO、大计算
- [ ] 高频回调（每帧/每 tick）中 new 对象、拼字符串 → GC 压力/卡顿
- [ ] 数据变化即全量重建列表/控件/位图，无增量更新
- [ ] 定时器高频轮询可由事件/回调驱动的状态

## Missing Idempotency 【在线】

- [ ] Payment/creation endpoints lack idempotency key — retry causes double-charge/double-creation
- [ ] Message consumers break on duplicate delivery — `at-least-once` semantics need idempotent handler
- [ ] No dedup key on outbound API calls — network retry sends the same webhook/email twice
- [ ] Cron jobs overlap — previous run hasn't finished before next trigger fires (no mutex/lock)

## Completion Criterion

Every shared mutable state has a clear ownership/leak policy. Locks have a defined order. Retry paths are idempotent.
本地项目：全库事件订阅已与退订配对核对（或确认生命周期相同无需退订）；Dispose 路径三件事齐全。

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D2:**
- Deadlock on critical path (payment, auth), UI 线程同步等待异步（死锁）→ CRITICAL
- Resource leak on long-running process, missing idempotency on payments, 高频路径内存泄露 → HIGH
- Missing lock ordering discipline, TOCTOU in non-critical path, 低频路径泄露 → MEDIUM
- Overlap in infrequent cron jobs → LOW
