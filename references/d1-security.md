# D1 — Security Audit Checklist

The dimension reviewers most often skip because "it compiles." Don't.

> **类型标注**：检查项标【通用】（每次必查）/【在线】（仅定位=在线时查）。
> 定位不符的项跳过，在报告中标注 N/A。混合项目按子项目定位过滤。

## Injection Attacks 【在线】

- [ ] SQL built from string concat/format instead of parameterized queries — `grep -r "execute.*f\""`, `grep -r "format.*SELECT"`
- [ ] Shell commands built from unsanitized user input — `grep -r "os\.system\|subprocess.*shell=True\|execSync\|child_process\.exec("`
- [ ] Template injection — f-strings/`.format()` inside Jinja/Mako/Handlebars contexts
- [ ] XPath / LDAP / NoSQL injection variants — string-built queries to non-SQL backends
- [ ] SSRF: user-supplied URL fetched server-side without allowlist
- [ ] 路径穿越：文件路径由用户输入拼接，未做 `..` 清洗与根目录约束
- [ ] 不受信任数据直接反序列化为可执行对象

## Authentication & Authorization 【在线】

- [ ] New route/mutation without auth middleware — check each endpoint definition
- [ ] IDOR: user A can access user B's resource by swapping IDs in the URL/body
- [ ] Privilege escalation: regular user hitting admin-only operations；只校验"已登录"未校验"角色/权限"的管理操作
- [ ] Session fixation: no session regeneration after login
- [ ] JWT: weak/HS256 with public-known secret, missing expiry, no issuer verification；token 过期/刷新/吊销逻辑；登出后 token 是否仍有效
- [ ] Weak password hashing (MD5/SHA1 instead of bcrypt/argon2/scrypt)
- [ ] 危险操作无防护：删除/覆盖数据无确认、无软删除、无审计日志

## Data Exposure

- [ ] Secrets/hardcoded credentials in source — `grep -rE "(api_key|secret|password|token|passwd)\s*=\s*['\"][^'\"]{6,}['\"]"` 【通用】
- [ ] Sensitive data in logs — `grep -r "console\.log\|print\|logger\."` near PII fields 【通用】
- [ ] 敏感数据（密码/token/密钥/PII）明文落库，应为哈希/加密 【在线】
- [ ] 内部服务间调用明文传输敏感字段 【在线】
- [ ] Stack traces returned to clients in production — check error handler config 【在线】
- [ ] Unencrypted transport — HTTP where HTTPS expected, `http://` in config, plaintext DB connections 【通用】
- [ ] PII in URL parameters (GET /users?id=123) vs POST body — URLs get logged everywhere 【在线】

## Web & Service Boundary 【在线】

- [ ] CORS: `Access-Control-Allow-Origin: *` + 允许凭证
- [ ] 限流缺失：昂贵端点（导出、搜索、发送）无限流，可被刷爆

## Input Validation 【通用】

- [ ] Missing size limits on uploads, batch operations, pagination (`limit=999999` from user)
- [ ] Type coercion traps — `"false" === true` in JS, `"0"` truthiness in Python, `==` vs `===`/`is`
- [ ] File upload: no content sniffing, no extension whitelist, no size cap, no mime validation
- [ ] CSV injection — user-generated cells starting with `=`, `+`, `-`, `@`
- [ ] Race condition in upload rename (TOCTOU between `exists()` and `rename()`)

## Dependencies 【通用】

- [ ] Known CVEs in pinned versions — `pip audit`, `npm audit`, `cargo audit`, `govulncheck`
- [ ] Unpinned transitive deps that silently change behavior — check lockfiles exist
- [ ] License conflicts — GPL code in proprietary product (run `license-check` if available)
- [ ] Deprecated/EOL packages in use；未使用的运行时依赖（拖累体积/攻击面）→ MEDIUM

## Completion Criterion

Every user-touched input path has a documented trust boundary. No secret appears in source. All deps are audited.
在线项目：每个对外端点已核对 鉴权 → 属主校验 → 输入校验；敏感字段已在存储、日志、响应中全库搜索。

## How to Report

Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D1:**
- Hardcoded secrets, SQL injection, IDOR on PII → CRITICAL
- Missing auth on non-sensitive endpoint, weak hashing, 明文敏感数据, CORS 配置错误 → HIGH
- Missing input size caps, unmigrated deprecated dependency → MEDIUM
- Minor input validation gaps → LOW
