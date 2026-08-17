# D2 — Concurrency & Resources Audit Checklist

Bugs that only manifest under load. Your test suite won't catch these.

## Race Conditions
- [ ] Read-modify-write without atomicity — `x += 1`, `counter++`, `collections.Counter[x] += 1` when `x` is shared across threads/async
- [ ] TOCTOU: checking a state, then acting on it — `if file.exists(): file.delete()`, `if os.path.exists(x): shutil.rmtree(x)`
- [ ] Check-then-act without lock — `if not in_dict[key]: in_dict[key] = compute()` 
- [ ] Missing distributed lock for shared-resource mutations across instances (e.g., two app servers both running the same cron job)
- [ ] File write without file lock — multiple workers writing the same log/cache file

## Deadlocks
- [ ] Lock ordering inconsistency — Thread A holds L1 → wants L2; Thread B holds L2 → wants L1
- [ ] Nested `synchronized`/`Lock`/`mutex`/`acquire()` without ordering discipline
- [ ] Holding a lock across an I/O or network call (DB query, HTTP fetch)
- [ ] `asyncio.Lock`/`threading.Lock` acquired but not released (missing `try/finally` or `async with`)

## Resource Leaks
- [ ] DB connections not returned to pool — no `try/finally`, no `with` context manager around `connection.cursor()`
- [ ] File handles left open — `open()` without `with` or `.close()` in finally
- [ ] Sockets / HTTP clients not closed — `aiohttp.ClientSession`, `urllib`, `requests.Session` lifecycle
- [ ] Event listeners attached without removal — `addEventListener` with no `removeEventListener`, React `useEffect` without cleanup, Vue `mounted` without `beforeDestroy`
- [ ] `setTimeout`/`setInterval` not cleared on component unmount
- [ ] Worker threads / goroutines spawned without shutdown mechanism (goroutine leak)
- [ ] Generators / iterators holding large resources open (`yield` blocks not closed)

## Missing Idempotency
- [ ] Payment/creation endpoints lack idempotency key — retry causes double-charge/double-creation
- [ ] Message consumers break on duplicate delivery — `at-least-once` semantics need idempotent handler
- [ ] No dedup key on outbound API calls — network retry sends the same webhook/email twice
- [ ] Cron jobs overlap — previous run hasn't finished before next trigger fires (no mutex/lock)

## Completion Criterion
Every shared mutable state has a clear ownership/leak policy. Locks have a defined order. Retry paths are idempotent.

## How to Report
Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D2:**
- Deadlock on critical path (payment, auth) → CRITICAL
- Resource leak on long-running process, missing idempotency on payments → HIGH
- Missing lock ordering discipline, potential TOCTOU in non-critical path → MEDIUM
- Overlap in infrequent cron jobs → LOW
