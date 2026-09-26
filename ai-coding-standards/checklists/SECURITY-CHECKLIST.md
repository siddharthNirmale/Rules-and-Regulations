# Mandatory Website Security Requirements & Verification Checklist

> **Purpose**: A comprehensive, production-grade security specification and verification checklist for modern web applications. Distinguishes between **Universally Applicable** controls (required on 100% of production workloads) and **Context-Dependent / Conditional** controls (triggered by specific architectural features).

---

## 1. Classification Framework

Every security control in this standard is categorized by its enforcement scope:

- **[UNIVERSAL]**: Non-negotiable baseline. Applies to all software regardless of stack, size, or hosting model.
- **[CONDITIONAL - AUTH/SESSIONS]**: Mandatory whenever user authentication, accounts, or persistent sessions exist.
- **[CONDITIONAL - DATA PERSISTENCE]**: Mandatory whenever databases (SQL/NoSQL) or file stores are used.
- **[CONDITIONAL - INTEGRATIONS/APIs]**: Mandatory when exposing public HTTP/JSON APIs or communicating with third-party web services.
- **[CONDITIONAL - USER CONTENT]**: Mandatory when handling file uploads, rich-text input, or user-supplied URLs.

---

## 2. Comprehensive Security Control Matrix

| ID | Control Area | Scope | Primary Threat Mitigated | Enforceable Standard & When Appropriate |
| :--- | :--- | :--- | :--- | :--- |
| **SEC-01** | **Authentication** | Conditional (Auth) | Account takeover, credential stuffing | Multi-step authentication, constant-time comparisons, password complexity. Required whenever user accounts exist. |
| **SEC-02** | **Authorization (RBAC/ABAC)** | Conditional (Auth) | Privilege escalation, IDOR, BOLA | Enforce authorization server-side on every request. Never rely on UI button hiding. Check tenant/user ownership on all queries. |
| **SEC-03** | **Session Management** | Conditional (Auth) | Session hijacking, session fixation | Regenerate session IDs upon login. Invalidate server-side on logout. Set absolute (e.g. 7d) and idle (e.g. 30m) timeouts. |
| **SEC-04** | **Password Security** | Conditional (Auth) | Offline dictionary/brute-force attacks | Hash passwords exclusively with Argon2id (m=64MB, t=3, p=4) or bcrypt (cost >= 12). Never use MD5/SHA256. |
| **SEC-05** | **Multi-Factor Auth (MFA)** | Conditional (Auth) | Compromised credential reuse | Require TOTP (RFC 6238) or FIDO2/WebAuthn for administrative accounts and sensitive operations. |
| **SEC-06** | **Input Validation** | Universal | Injection, buffer overflow, ReDoS | Validate type, length, range, format using strict schemas (Zod/Pydantic). Reject unexpected fields. |
| **SEC-07** | **Output Encoding** | Universal | Cross-Site Scripting (XSS) | Context-aware encoding (HTML entity, JS attribute, URL encoding) before rendering untrusted data into markup. |
| **SEC-08** | **XSS Prevention** | Universal | Client script injection, cookie theft | Sanitize dynamic markup with DOMPurify. Avoid `dangerouslySetInnerHTML`. Implement strict Content Security Policy. |
| **SEC-09** | **CSRF Protection** | Conditional (Cookie) | Unauthorized state-changing actions | SameSite=Lax/Strict cookies + anti-CSRF synchronizer tokens on state-changing requests (POST/PUT/PATCH/DELETE). |
| **SEC-10** | **SQL/NoSQL Injection** | Conditional (DB) | Data theft, database destruction | Use parameterized queries / prepared statements exclusively. Never interpolate raw strings into query builders or ORMs. |
| **SEC-11** | **Command Injection** | Universal | Remote Code Execution (RCE) on host | Avoid OS shell execution (`exec`). Use `execFile` with segregated argument arrays. Sanitize and allowlist all arguments. |
| **SEC-12** | **SSRF Prevention** | Conditional (Fetch) | Internal cloud metadata theft (AWS/GCP) | Validate user-supplied URLs. Resolve DNS and block private (RFC 1918), loopback (`127.0.0.1`), and cloud metadata (`169.254.169.254`) IPs. |
| **SEC-13** | **Path Traversal** | Universal | System file disclosure (`/etc/passwd`) | Strip directory separators (`..`, `/`, `\`). Use `path.basename()`. Resolve absolute paths and assert base-directory containment. |
| **SEC-14** | **Secure File Uploads** | Conditional (Files) | Malicious executable upload, DoS | Validate MIME types via magic bytes (not file extensions). Store outside web root (e.g. S3). Randomize filenames. Enforce max file size. |
| **SEC-15** | **API Security** | Conditional (APIs) | Broken object level auth, data scrape | Authenticate every API route. Apply request body limits. Return standardized error envelopes without stack traces. |
| **SEC-16** | **Rate Limiting** | Universal | DoS, credential stuffing, scraping | Enforce IP- and user-based token bucket or sliding window rate limits (e.g. 5 attempts/min on auth, 100 req/min on APIs). |
| **SEC-17** | **Brute-Force Protection** | Conditional (Auth) | Password guessing, OTP enumeration | Exponential backoff or progressive delays on failed login attempts. Implement CAPTCHA or temporary account lock after 5 failures. |
| **SEC-18** | **CORS Configuration** | Universal | Cross-origin data theft | Never use `Access-Control-Allow-Origin: *` with credentials. Explicitly allowlist exact trusted domains. |
| **SEC-19** | **Security Headers** | Universal | Clickjacking, MIME-sniffing, downgrade | Send `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`. |
| **SEC-20** | **Content Security Policy**| Universal | XSS mitigation, data exfiltration | Define strict CSP: `default-src 'self'`. Disallow `'unsafe-inline'` and `'unsafe-eval'`. Use nonces for dynamic scripts. |
| **SEC-21** | **HTTPS / TLS Configuration**| Universal | Man-in-the-middle (MitM) eavesdropping | Enforce TLS 1.3 (or 1.2 minimum). Send `Strict-Transport-Security` (HSTS) header with `max-age=63072000; includeSubDomains; preload`. |
| **SEC-22** | **Secure Cookies** | Universal | Session hijacking, cookie interception | All auth and session cookies MUST have `HttpOnly; Secure; SameSite=Lax` (or `Strict`). Prefix sensitive cookies with `__Host-`. |
| **SEC-23** | **Secrets Management** | Universal | Credential compromise via version control | Never commit API keys, tokens, or passwords to git. Inject secrets via environment variables or secret vaults (Vault, AWS Secrets Manager). |
| **SEC-24** | **Environment Variables** | Universal | Misconfiguration crashes, exposed tokens | Validate environment variables at boot with strict schemas. Expose only non-sensitive variables to frontend client bundles. |
| **SEC-25** | **Dependency Security** | Universal | Supply-chain attacks, vulnerable deps | Commit lockfiles (`package-lock.json`, `pnpm-lock.yaml`). Run `npm audit` / `cargo audit` in CI. Pin exact dependency versions. |
| **SEC-26** | **Supply-Chain Security** | Universal | Malicious package injection (typosquatting) | Enforce automated SCA (Software Composition Analysis) like Dependabot or Snyk. Review dependencies with zero maintenance. |
| **SEC-27** | **Error-Message Security**| Universal | Information leakage, architecture mapping| Never display stack traces, database error codes, or SQL fragments to public clients. Return generic error messages with tracking IDs. |
| **SEC-28** | **Logging & Monitoring** | Universal | Undetected intrusion, compliance breach | Log all auth failures, permission rejections, and admin mutations. Redact PII, passwords, and tokens from logs. Ship logs off-server. |
| **SEC-29** | **Sensitive Data Protection**| Universal | Identity theft, compliance penalties | Encrypt PII, tokens, and sensitive business data at rest (AES-256-GCM). Mask credit card numbers and personal identifiers in UI. |
| **SEC-30** | **Privacy Considerations**| Universal | Regulatory fines (GDPR, CCPA) | Minimize data collection. Provide mechanisms for account/data deletion. Disclose cookie usage and secure user consent. |
| **SEC-31** | **Secure Database Access**| Conditional (DB) | Lateral movement after DB breach | Connect over TLS. Grant database users least privilege (e.g. app user cannot `DROP TABLE`). Store connection strings in secrets. |
| **SEC-32** | **Backup & Disaster Recovery**| Universal | Ransomware, catastrophic data loss | Automated daily encrypted database backups. Test restoration procedures quarterly. Store off-site with immutable retention locks. |
| **SEC-33** | **Production Configuration**| Universal | Debug interface exposure, defaults | Disable debug modes (`DEBUG=false`, `NODE_ENV=production`). Remove default accounts, test routes, and Swagger UI in public production. |
| **SEC-34** | **Vulnerability Scanning** | Universal | Unpatched zero-day vulnerabilities | Integrate automated DAST (OWASP ZAP) and SAST (Semgrep, SonarQube) scans into the CI/CD deployment pipeline. |

---

## 3. Implementation Deep Dives & Concrete Guardrails

### 3.1 Security Headers Configuration
Every web server or gateway (Express, Next.js, Nginx, Caddy, Cloudflare) **MUST** respond with the following baseline headers:

```http
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none';
```

### 3.2 Secure Cookie Recipe
When issuing authentication or session cookies:

```typescript
res.cookie('__Host-session_id', token, {
  httpOnly: true,                               // Inaccessible to document.cookie (defeats XSS theft)
  secure: process.env.NODE_ENV === 'production', // Transmitted strictly over HTTPS
  sameSite: 'lax',                              // Defeats cross-site request forgery
  path: '/',                                    // Required for __Host- prefix
  maxAge: 7 * 24 * 60 * 60 * 1000               // 7-day finite lifetime
});
```

### 3.3 SSRF Defense Guardrail
Whenever fetching a remote URL supplied by an end-user or webhook:

```typescript
import dns from 'node:dns/promises';
import ipaddr from 'ipaddr.js';

export async function assertSafeUrl(urlString: string): Promise<URL> {
  const url = new URL(urlString);

  // 1. Protocol allowlist
  if (url.protocol !== 'http:' && url.protocol !== 'https:') {
    throw new SecurityError(`Forbidden protocol: ${url.protocol}`);
  }

  // 2. Resolve DNS to verify IP
  const addresses = await dns.lookup(url.hostname, { all: true });
  for (const { address } of addresses) {
    const ip = ipaddr.parse(address);
    // Block RFC 1918 private, link-local, loopback, and cloud metadata
    if (ip.range() !== 'unicast' || address === '169.254.169.254') {
      throw new SecurityError(`SSRF Hazard: Destination IP ${address} is not a public routable address`);
    }
  }

  return url;
}
```

### 3.4 Path Traversal Defense Guardrail
Whenever reading, saving, or serving files from disk:

```typescript
import path from 'node:path';

export function resolveSafePath(baseDirectory: string, userInputFilename: string): string {
  // Strip null bytes and extract base name
  const cleanFilename = path.basename(userInputFilename.replace(/\0/g, ''));
  const safeAbsolutePath = path.resolve(baseDirectory, cleanFilename);

  // Assert target path resides strictly inside intended root directory
  if (!safeAbsolutePath.startsWith(path.resolve(baseDirectory))) {
    throw new SecurityError('Path traversal attempt detected');
  }

  return safeAbsolutePath;
}
```

---

## 4. Verification & Audit Protocol

An AI agent or human auditor **MUST** verify compliance before production deployment:

- [ ] **Secret Scan**: Run `git grep -iE 'api[_-]?key|secret|password|bearer'` -> Zero hardcoded credentials found.
- [ ] **Query Audit**: All SQL/NoSQL queries inspected -> 100% parameterized. Zero string interpolation.
- [ ] **Header Inspection**: `curl -I https://staging.domain.com` -> All 6 mandatory security headers present.
- [ ] **Auth Enforcement**: Every protected API route returns `401 Unauthorized` without credentials.
- [ ] **Tenant Isolation**: User A cannot fetch User B's resources using User B's ID (`404` or `403` returned).
- [ ] **Audit Logs**: Failed logins and access denials are captured in structured audit logs.
