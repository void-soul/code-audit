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

## Schema & Migration Design 【通用】

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

## Schema Design Rationality 【通用｜适用条件：项目存在数据库】

> 无数据库的项目（纯文件/内存存储）本节整体标 N/A。

- [ ] **主键设计**：代理主键与自然主键混用；复合主键缺索引；主键在业务上可变（改主键=改外键链）
- [ ] **字段类型**：字符串存数字/日期；金额用 `float`（→ D7）；时间戳 UTC 与本地时区混存；布尔用 int 存 'Y/N'/'0/1' 混乱
- [ ] **可空性**：全表列都 NOT NULL 或全可空（丢失语义信息）；可空列无默认值导致插入报错
- [ ] **约束缺失**：状态/类型字段无 CHECK/ENUM；金额无精度约束；缺唯一约束导致重复数据
- [ ] **范式与冗余**：无节制冗余（同一事实存多份且无同步机制）；过度范式导致 5+ join
- [ ] **索引设计**：外键列无索引；组合索引列顺序不符查询；索引过多拖慢写入
- [ ] **审计字段缺失/不统一**：`created_at` vs `create_time` vs `createdAt` 混用；updated_at 无自动更新
- [ ] **软删除无全局策略**：有 `is_deleted` 但查询未统一过滤 → 漏加条件即数据错乱
- [ ] **命名规范混用**：camelCase 与 snake_case 混用；单复数混用；表前缀策略不一致
- [ ] **迁移管理缺失**：直接改生产库；迁移不可回滚（→ Schema Design 节）
- [ ] **大表/热点行设计**：单表千万级且频繁更新同一行 → 行锁热点
- [ ] **多语言/多币种值直接存字段列**（应规范化）

**严重度**：设计缺陷导致数据不一致或丢失=CRITICAL；审计字段/命名/索引问题=MEDIUM。

---

## Dead Tables & Dead Columns（僵尸表/僵尸字段）【通用｜适用条件：项目存在数据库】

- [ ] **僵尸表**：schema 中存在但代码中无任何读写（无 SELECT/INSERT/UPDATE/DELETE、无 ORM 实体映射、无视图/存储过程引用）
- [ ] **僵尸字段**：列已定义但代码从不读写（仅靠 ORM 自动映射不算"使用"）
- [ ] **历史遗留表**：旧功能下线后表未删（改功能未清 schema）
- [ ] **影子表**：无写入来源的日志/统计/中间表
- [ ] **仍在写入但无人读取的表** → 数据风险（敏感数据长期沉积）=HIGH
- [ ] **误报排除**：审计表、迁移历史表、框架内置表、只读视图、BI/运维脚本手工使用的表 —— 判定前需向用户确认，不可直接判死
- [ ] **检测方法**：
  1. 从迁移脚本 / DDL 列出全部表与列（schema 真源优先级：迁移文件 > 实体定义 > `schema.sql` > 代码内 `CREATE TABLE`）
  2. 全库搜索表名/列名出现处，区分「DDL 声明」与「实际使用」
  3. ORM：实体属性在业务代码中无任何引用 → 僵尸字段候选
  4. 双向差集结果需人工确认后再列为发现项
- [ ] **附带收益**：清理僵尸表/字段可同时解决 D4 的"过度范式/索引过多/约束冗余"

**严重度**：僵尸表=MEDIUM（累积运维与认知成本）；僵尸字段=LOW；仍写入无人读的敏感表=HIGH。

---

## Code–Schema Drift（代码使用了不存在的表/字段）【通用｜适用条件：项目存在数据库】

- [ ] 代码引用了 schema 中**不存在的表/视图**（拼写错误、已删除表、表名大小写/前缀不一致）
- [ ] 代码引用了**不存在的列**：SELECT 列表、WHERE 条件、INSERT 列清单、UPDATE SET、ORM 字符串列名（裸串、插值 SQL、raw SQL、query builder）
- [ ] ORM 实体属性与 DB 列名/类型/可空性不匹配（迁移改了库、实体没同步）
- [ ] 迁移脚本与实体定义漂移：新环境能跑、老环境运行期报错
- [ ] 表/列重命名后代码未同步（旧名在代码、旧名在新迁移中已消失）
- [ ] 隐性失败型：SQLite/MySQL 等不校验列名直到执行才报错 → 潜伏到运行时
- [ ] **双向差集检测法**（必须做，两个方向都要）：
  - `schema 有、代码无` → 僵尸表/僵尸字段（见上一节）
  - `代码有、schema 无` → **漂移（本节）**，是运行期 SQL 错误/数据写错列的直接来源
- [ ] 覆盖动态 SQL、字符串拼接的列名、`nameof` 之外的裸标识符

**严重度**：漂移（代码引用不存在的表/列）=CRITICAL（运行期失败或写错列）；重命名未同步=HIGH。

---

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
存在数据库时：已产出表/列清单，且完成**双向差集**——schema 有代码无（僵尸）与代码有 schema 无（漂移）各出一份结论。

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D4:**
- Money inconsistency under race condition, missing FK causing data corruption, 代码引用不存在的表/列 → CRITICAL
- N+1 on high-traffic endpoint, missing transaction boundary for payment, 仍在写入却无人读取的敏感表 → HIGH
- Missing indexes on slow queries (p99 > 200ms), cache key collisions, 僵尸表, schema 设计缺陷（类型/可空性/约束缺失）→ MEDIUM
- Over-normalized schema on rarely-queried data, missing rollback scripts, 僵尸字段 → LOW
