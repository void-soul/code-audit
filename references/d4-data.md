# D4 — Data & Storage Audit Checklist

Where correctness meets durability. The "seems fine until it isn't" layer.

## Query Correctness
- [ ] N+1 queries — loop calling DB inside ORM iteration (`for item in query: item.related.find()`)
- [ ] Missing indexes on WHERE / JOIN / ORDER BY columns — check EXPLAIN plans if available
- [ ] SELECT * when only 2 columns are needed (over-fetch + coupling to schema)
- [ ] Implicit type coercion defeating indexes — `WHERE varchar_col = 123`, `WHERE int_col = '123'`
- [ ] Pagination without ORDER BY — inconsistent results across pages
- [ ] Large result sets fetched into memory without streaming/cursor

## Transactions & Consistency
- [ ] Operations that should be atomic but aren't — deduct balance + create ledger row split across transactions
- [ ] Missing optimistic locking on concurrent edits (`version` column or `updated_at` check)
- [ ] Pessimistic locking misused — `SELECT FOR UPDATE` on read-heavy paths causing contention
- [ ] Read-after-write inconsistency across replicas with no TTL guard (write master, read slave, data not yet replicated)
- [ ] Missing unique constraints allowing duplicate inserts under concurrency

## Schema Design
- [ ] Missing foreign keys allowing orphaned rows — `ON DELETE CASCADE` missing or wrong
- [ ] No migration rollback scripts — `up` without `down`/`reverse`
- [ ] `NOT NULL` columns added without default to existing tables (locks table on deploy)
- [ ] `NOT NULL` coercion for optional fields — frontend sends missing, backend crashes or silently defaults
- [ ] Money stored as `float`/`double` instead of `Decimal`/integer cents
- [ ] Over-normalized schema requiring 5+ joins for common queries (performance trap)

## Caching
- [ ] Cache invalidation strategy absent or ad hoc — no clear ownership of cached data
- [ ] Stale cache serving after writes with no TTL/bypass — write-through missing
- [ ] Cache key collisions — `cache.get(user_id)` where user_id overlaps across types
- [ ] Cache stampede/thundering herd — many requests miss cache simultaneously and all hit DB
- [ ] No TTL set (or TTL too long) on cached data that changes frequently

## Completion Criterion
Every transaction has a clear consistency boundary. Schema changes are reversible. Cache has an explicit invalidation path.

## How to Report
Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D4:**
- Money inconsistency under race condition, missing FK causing data corruption → CRITICAL
- N+1 on high-traffic endpoint, missing transaction boundary for payment → HIGH
- Missing indexes on slow queries (p99 > 200ms), cache key collisions → MEDIUM
- Over-normalized schema on rarely-queried data, missing rollback scripts → LOW
