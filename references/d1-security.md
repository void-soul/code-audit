# D1 — Security Audit Checklist

The dimension reviewers most often skip because "it compiles." Don't.

## Injection Attacks
- [ ] SQL built from string concat/format instead of parameterized queries — `grep -r "execute.*f\""`, `grep -r "format.*SELECT"`
- [ ] Shell commands built from unsanitized user input — `grep -r "os\.system\|subprocess.*shell=True\|execSync\|child_process\.exec("`
- [ ] Template injection — f-strings/`.format()` inside Jinja/Mako/Handlebars contexts
- [ ] XPath / LDAP / NoSQL injection variants — string-built queries to non-SQL backends
- [ ] SSRF: user-supplied URL fetched server-side without allowlist

## Authentication & Authorization
- [ ] New route/mutation without auth middleware — check each endpoint definition
- [ ] IDOR: user A can access user B's resource by swapping IDs in the URL/body
- [ ] Privilege escalation: regular user hitting admin-only operations
- [ ] Session fixation: no session regeneration after login
- [ ] JWT: weak/HS256 with public-known secret, missing expiry, no issuer verification
- [ ] Weak password hashing (MD5/SHA1 instead of bcrypt/argon2/scrypt)

## Data Exposure
- [ ] Secrets/hardcoded credentials in source — `grep -rE "(api_key|secret|password|token|passwd)\s*=\s*['\"][^'\"]{6,}['\"]"`
- [ ] Sensitive data in logs — `grep -r "console\.log\|print\|logger\."` near PII fields
- [ ] Stack traces returned to clients in production — check error handler config
- [ ] Unencrypted transport — HTTP where HTTPS expected, `http://` in config, plaintext DB connections
- [ ] PII in URL parameters (GET /users?id=123) vs POST body — URLs get logged everywhere

## Input Validation
- [ ] Missing size limits on uploads, batch operations, pagination (`limit=999999` from user)
- [ ] Type coercion traps — `"false" === true` in JS, `"0"` truthiness in Python, `==` vs `===`/`is`
- [ ] File upload: no content sniffing, no extension whitelist, no size cap, no mime validation
- [ ] CSV injection — user-generated cells starting with `=`, `+`, `-`, `@`
- [ ] Race condition in upload rename (TOCTOU between `exists()` and `rename()`)

## Dependencies
- [ ] Known CVEs in pinned versions — `pip audit`, `npm audit`, `cargo audit`, `govulncheck`
- [ ] Unpinned transitive deps that silently change behavior — check lockfiles exist
- [ ] License conflicts — GPL code in proprietary product (run `license-check` if available)
- [ ] Deprecated/EOL packages in use

## Completion Criterion
Every user-touched input path has a documented trust boundary. No secret appears in source. All deps are audited.

## How to Report
Record: `file:line → issue → fix | severity: critical/high/medium/low | effort: small/medium/large`

**Severity guide for D1:**
- Hardcoded secrets, SQL injection, IDOR on PII → CRITICAL
- Missing auth on non-sensitive endpoint, weak hashing → HIGH
- Missing input size caps, unmigrated deprecated dependency → MEDIUM
- Minor input validation gaps → LOW
