---
name: security-scan
description: Use when performing a security-focused review of the codebase — covers OWASP Top 10 checks, secrets scanning with truffleHog or gitleaks, and dependency audits via pip-audit or npm audit.
---

# Security Scan

## The Law
**Security issues found before a merge cost minutes to fix; the same issue found in production costs days — run the full scan every time, not just when it feels risky.**

## When to Use
- Before merging a branch that introduces new API endpoints, auth changes, or file handling
- During a periodic security audit of an existing codebase
- After adding a batch of new dependencies
- When onboarding to a codebase to understand its exposure surface
- **Never skip when:** the code handles user authentication, payment data, file uploads, inter-service tokens, or personally identifiable information — these categories require a full scan, not a spot check

## Process

### Phase 1: Secrets Scan
Goal — confirm no credentials, tokens, or private keys are committed.

Using gitleaks (preferred — scans full git history):
```bash
# Install
brew install gitleaks         # macOS
pip install gitleaks           # or via pip wrapper

# Scan the full repo history
gitleaks detect --source . --report-format json --report-path gitleaks-report.json

# Scan only the current diff (CI mode)
gitleaks protect --staged
```

Using truffleHog (deep entropy + regex scan):
```bash
pip install truffleHog
trufflehog git file://. --only-verified
```

Manual grep as a fast supplement:
```bash
grep -rn --include="*.env" --include="*.json" --include="*.yaml" \
  -E "(api_key|secret|password|private_key|token)\s*=" . \
  | grep -v ".git"
```

What to look for:
1. Hardcoded API keys in source files, config files, and test fixtures
2. Credentials in `.env` files committed to the repo (check `.gitignore` covers them)
3. Private keys or certificates stored in the repository
4. Passwords embedded in database connection strings in config files

If any are found: rotate the credential immediately before pushing a fix — the secret is already compromised.

### Phase 2: Dependency Audit
Goal — identify known vulnerabilities in third-party packages.

Node / TypeScript:
```bash
npm audit --audit-level=high
# Automated fix attempt (review before accepting)
npm audit fix

# Deeper audit with full advisory details
npx audit-ci --high
```

Python:
```bash
pip install pip-audit
pip-audit

# With a specific requirements file
pip-audit -r requirements.txt

# JSON output for CI parsing
pip-audit --output json > pip-audit-report.json
```

Triage process:
1. Fix any **critical** or **high** severity vulnerabilities before continuing — these are blockers
2. For **moderate** vulnerabilities, check whether the vulnerable code path is actually reachable
3. For **low** vulnerabilities, document and schedule a fix — they are not blockers but must not be ignored indefinitely
4. If a fix is not available, document the exposure and apply a compensating control (WAF rule, input restriction, feature flag)

### Phase 3: OWASP Top 10 Checklist
Goal — manually verify the ten most common vulnerability classes.

Work through each category for the codebase in scope:

**A01 — Broken Access Control**
1. Confirm every endpoint checks that the authenticated user has permission for the requested resource
2. Verify that horizontal privilege escalation is impossible (user A cannot access user B's data by changing an ID in a request)
3. Check that directory listing is disabled on static file servers

**A02 — Cryptographic Failures**
4. Confirm passwords are hashed with bcrypt, Argon2, or scrypt — never MD5, SHA-1, or unsalted SHA-256
5. Verify TLS is enforced on all connections that carry sensitive data
6. Check that sensitive data is not logged in plaintext (PII, card numbers, passwords)

**A03 — Injection**
7. Confirm all database queries use parameterised statements or ORM query builders — no string concatenation into SQL
8. Check for template injection: user input must not reach a template engine unescaped
9. Verify shell commands do not include user-controlled input; use argument arrays, not shell strings

TypeScript — parameterised query examples:
```typescript
// Red: string interpolation into SQL
const rows = await db.query(`SELECT * FROM users WHERE id = ${userId}`);

// Green: parameterised
const rows = await db.query("SELECT * FROM users WHERE id = $1", [userId]);
```

Python — equivalent:
```python
# Red: f-string into query
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# Green: parameterised
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

**A04 — Insecure Design**
10. Confirm that rate limiting is applied to authentication endpoints
11. Verify that account enumeration is not possible through differing error messages for valid vs invalid usernames
12. Check that sensitive operations (password reset, email change) require re-authentication

**A05 — Security Misconfiguration**
13. Confirm debug mode and verbose error responses are disabled in production config
14. Verify that default credentials have been changed on all services
15. Check that CORS policy is restrictive — `*` origin is only acceptable for fully public APIs

**A06 — Vulnerable and Outdated Components**
16. Dependency audit output from Phase 2 covers this — confirm it was run and reviewed

**A07 — Identification and Authentication Failures**
17. Confirm session tokens are sufficiently random and have appropriate expiry
18. Verify multi-factor authentication is available for sensitive accounts
19. Check that failed login attempts are rate-limited and logged

**A08 — Software and Data Integrity Failures**
20. Confirm dependency integrity is verified (package-lock.json or poetry.lock committed and pinned)
21. Verify that CI pipeline steps cannot be hijacked by third-party actions without pinned SHA versions

**A09 — Security Logging and Monitoring Failures**
22. Confirm authentication events (login, logout, failure, password reset) are logged with timestamp and user identifier
23. Verify logs are written to a location that cannot be modified by the application itself

**A10 — Server-Side Request Forgery (SSRF)**
24. Confirm that any URL accepted from a user is validated against an allowlist before the server fetches it
25. Verify that internal network addresses (10.x, 172.16.x, 169.254.x) are blocked from user-supplied URLs

### Phase 4: Report
Goal — produce a prioritised list of findings.

Structure each finding as:
- **Severity:** Critical / High / Moderate / Low
- **Category:** OWASP category or scan type
- **Location:** file path and line number
- **Description:** what the vulnerability is and how it could be exploited
- **Recommendation:** the specific change needed to resolve it

Severity definitions:
- **Critical** — exploitable with no authentication required, direct data access or code execution
- **High** — exploitable with user-level access, significant data exposure
- **Moderate** — requires specific conditions or chaining with another issue
- **Low** — defence-in-depth improvement with limited direct exploitability

## Red Flags — Stop Immediately
- Any hardcoded credential or private key in the source tree — rotate it before fixing the code
- SQL query built by concatenating user input with a string
- A `*.env` file committed to git history — rewrite the history or accept that the secret is compromised
- An authentication endpoint with no rate limiting
- CORS set to `Access-Control-Allow-Origin: *` on an endpoint that returns user data

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "We're internal only, no need for a full scan" | Internal tools are frequently the first pivot point in a breach |
| "The dependency vulnerability doesn't affect our code path" | Attackers exploit vulnerabilities in ways developers don't anticipate |
| "We'll fix the hardcoded key after the demo" | The key is already exposed the moment it is committed |
| "Rate limiting is the ops team's job" | Application-layer rate limiting must exist independent of infrastructure controls |
| "Our ORM prevents injection" | ORMs with raw query escape hatches are still vulnerable when those hatches are used |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| Secrets Scan | gitleaks or truffleHog + manual grep | Zero credentials in repo or history |
| Dependency Audit | `npm audit` / `pip-audit` | No critical or high vulnerabilities unaddressed |
| OWASP Top 10 | Work through all 10 categories | Each checklist item confirmed or finding logged |
| Report | Prioritised findings with location and fix | Every critical/high item has a resolution plan |
