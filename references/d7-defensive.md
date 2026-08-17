# D7 — Defensive Details Audit Checklist

The small things that become big 5am pages. Numeric precision, time, magic constants, recursion.

## Hardcoded Values
- [ ] Magic numbers/strings without named constants — `if status == 3` (what is 3?), `timeout = 30` (30 what? seconds? minutes?)
- [ ] IPs, ports, and domain names in code instead of config — `http://10.0.0.5:8080/api` hardcoded
- [ ] Pagination defaults hardcoded at call sites — `{ limit: 20 }` scattered everywhere instead of `DEFAULT_PAGE_SIZE`
- [ ] Retry/backoff parameters hardcoded — `sleep(5)` instead of `EXPONENTIAL_BACKOFF_INITIAL`
- [ ] Timeout values missing from external calls — `requests.get(url)` with no `timeout=` kwarg (hangs forever)

## Date & Time
- [ ] Naive datetime compared across timezones — `datetime.now()` vs `datetime.now(tz)` when users are global
- [ ] Server-local time stored without timezone — `new Date().toISOString()` in JS but server moves to different TZ
- [ ] Floating holidays (Lunar New Year, US Thanksgiving) handled with date arithmetic that doesn't account for them
- [ ] Leap year / leap second / DST transition edge cases
- [ ] Millisecond vs microsecond vs nanosecond precision confusion — `Date.now()` (ms) vs `time.time()` (s) vs `Instant.now()` (ns)
- [ ] Time comparison with floating point — `if (timeout - elapsed > 0.001)` unreliable with float math

## Numeric
- [ ] Money as `float`/`double` — `price = 19.99` in float, accumulate 1000 of them and cents are off
- [ ] `==` comparison on floats — `if (balance == 0.0)`, `if (totalAmount === 0.1 + 0.2)` in JS
- [ ] Integer overflow on large IDs, counters, timestamps (Year 2038 problem for 32-bit)
- [ ] Division by zero guards missing — `total / count` when count may be 0
- [ ] Floating point equality in map keys / Set membership — `Set([0.1 + 0.2])` doesn't contain `0.3`
- [ ] Rounding mode unspecified — `0.5` rounds to `0` or `1`? Banker's rounding vs arithmetic rounding

## Recursion
- [ ] Recursive traversal of user-controlled depth without a limit — `renderTree(node.children)` where user can nest 10000 deep
- [ ] No base case visible or base case unreachable due to an off-by-one bug
- [ ] Mutual recursion (A calls B, B calls A) without a termination guarantee
- [ ] Stack overflow risk deep enough to crash the process (no tail-call optimization in most languages)

## Completion Criterion
No magic constants. All arithmetic on money uses safe numeric types. All datetime operations are timezone-explicit. Recursive depth is bounded.

## How to Report
Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D7:**
- Money arithmetic with float on financial totals, missing division-by-zero guard on critical computation → CRITICAL
- Naive timezone comparison on user-facing timestamps, float equality on payment amounts → HIGH
| Magic numbers on non-critical paths, missing named constants → MEDIUM
| Unnamed pagination default | LOW
