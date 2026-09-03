# D4 — Data & Storage Audit Checklist

Where correctness meets durability. The "seems fine until it isn't" layer.

> **类型标注**：【通用】（每次必查）/【在线】（仅定位=在线时查）。
> 定位不符的项跳过，在报告中标注 N/A。混合项目按子项目定位过滤。

## Query Correctness 【通用】

- [ ] N+1 queries — loop calling DB inside ORM iteration (`for item in query: item.related.find()`)
- [ ] Missing indexes on WHERE / JOIN / ORDER BY columns — check EXPLAIN plans if available
- [ ] SELECT * when only 2 columns are needed (over-fetch + coupling to schema)
- [ ] Implicit type coercion defeating indexes — `WHERE varchar_col = 123`, `WHERE int_col = '123'`
- [ ] Pagination without ORDER BY — inconsistent results across pages
- [ ] 无分页全量查询：`findAll()`/`SELECT *` 无 LIMIT 全表加载进内存
- [ ] Large result sets fetched into memory without streaming/cursor

## Transactions & Consistency 【通用】

- [ ] Operations that should be atomic but aren't — deduct balance + create ledger row split across transactions
- [ ] Missing optimistic locking on concurrent edits (`version` column or `updated_at` check)
- [ ] Pessimistic locking misused — `SELECT FOR UPDATE` on read-heavy paths causing contention
- [ ] Read-after-write inconsistency across replicas with no TTL guard (write master, read slave, data not yet replicated)
- [ ] Missing unique constraints allowing duplicate inserts under concurrency

## Schema Design 【通用】

- [ ] Missing foreign keys allowing orphaned rows — `ON DELETE CASCADE` missing or wrong
- [ ] No migration rollback scripts — `up` without `down`/`reverse`
- [ ] `NOT NULL` columns added without default to existing tables (locks table on deploy)
- [ ] `NOT NULL` coercion for optional fields — frontend sends missing, backend crashes or silently defaults
- [ ] Money stored as `float`/`double` instead of `Decimal`/integer cents
- [ ] Over-normalized schema requiring 5+ joins for common queries (performance trap)

## Caching 【通用】

- [ ] Cache invalidation strategy absent or ad hoc — no clear ownership of cached data
- [ ] Stale cache serving after writes with no TTL/bypass — write-through missing
- [ ] Cache key collisions — `cache.get(user_id)` where user_id overlaps across types
- [ ] Cache stampede/thundering herd — many requests miss cache simultaneously and all hit DB
- [ ] No TTL set (or TTL too long) on cached data that changes frequently
- [ ] 重复昂贵计算无缓存层：同一请求/多请求重复计算可缓存结果

## Service Performance 【在线】

- [ ] 请求线程同步阻塞：请求处理线程内做同步 IO（文件、外部 HTTP 无超时）
- [ ] 外部调用无 timeout / 重试无上限无退避 —— 上游 hang 住拖垮调用方
- [ ] 大事务：事务内做外部 HTTP 调用/重计算，长时间持锁
- [ ] 无边界内存队列：慢消费时 OOM
- [ ] 对下游并发请求数无信号量限制
- [ ] 大对象全量序列化后返回/写日志（应投影、分页、流式）
- [ ] 启动路径昂贵操作：服务启动时同步全量数据加载（拖慢部署/重启）

**严重度**：全表加载进内存/大事务持锁=HIGH；N+1=HIGH（热路径）/MEDIUM（冷路径）；无超时=HIGH；缺缓存=MEDIUM。

## Completion Criterion

Every transaction has a clear consistency boundary. Schema changes are reversible. Cache has an explicit invalidation path.
在线项目：高频数据访问路径已核对分页/索引/缓存；全部外部调用已核对 timeout 与重试上限。

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D4:**
- Money inconsistency under race condition, missing FK causing data corruption → CRITICAL
- N+1 on high-traffic endpoint, missing transaction boundary for payment → HIGH
- Missing indexes on slow queries (p99 > 200ms), cache key collisions → MEDIUM
- Over-normalized schema on rarely-queried data, missing rollback scripts → LOW
