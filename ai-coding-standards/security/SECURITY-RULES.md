# Core Security Rules & Defense-in-Depth

> **Scope**: Foundational security architecture, OWASP ASVS Level 2, defense-in-depth, and severe attack vector countermeasures.

---

## The Zero Trust Security Principle
No request, network packet, cookie, or API payload is inherently trustworthy. Whether an HTTP request originates from an external browser, an internal microservice, or an automated cron job, identity and permissions MUST be established explicitly on every single call.

---

## Rule SEC-01: Zero Secrets in Version Control [MUST]
- **Rule**: Source code, configuration files, scripts, commit messages, and test mocks MUST NEVER contain live API keys, tokens, passwords, private keys, or credentials.
- **Reason**: Version control repositories are duplicated across developer laptops, CI runners, and third-party hosting services. A committed credential is permanently compromised.
- **Risk**: Automated bots scan GitHub within 30 seconds of push, exfiltrating AWS keys, database passwords, and Stripe tokens.
- **Correct Approach**:
  - Load all secrets dynamically from validated environment variables or cloud secrets managers (AWS Secrets Manager, HashiCorp Vault).
  - Use `.env.example` containing only variable keys and non-sensitive placeholder comments.
  - Implement pre-commit hooks using `gitleaks` or `trufflehog`.
- **Exceptions**: Public dummy test keys explicitly designated as mock strings in test suites (e.g. `pk_test_placeholder`).

---

## Rule SEC-02: Mandatory HTTP Security Headers [MUST]
- **Rule**: Every production web application MUST return the complete suite of standard security headers on all responses:
  1. `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`
  2. `X-Content-Type-Options: nosniff`
  3. `X-Frame-Options: DENY` (or `SAMEORIGIN`)
  4. `Referrer-Policy: strict-origin-when-cross-origin`
  5. `Permissions-Policy: camera=(), microphone=(), geolocation=()`
  6. `Content-Security-Policy`: Disallowing `'unsafe-inline'` and `'unsafe-eval'`.
- **Reason**: Security headers instruct modern browsers to enforce strict sandboxing, prevent clickjacking, block MIME-type sniffing, and enforce HTTPS encryption.
- **Risk**: Clickjacking, downgrade attacks, and script injection.
- **Correct Approach**: Configure Helmet in Node.js or apply headers at the reverse proxy (Nginx/Cloudflare):
  ```typescript
  import helmet from 'helmet';
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", 'data:', 'https:'],
        objectSrc: ["'none'"],
        upgradeInsecureRequests: []
      }
    }
  }));
  ```
- **Exceptions**: Embeddable widgets explicitly intended to be framed inside customer iframes (requires specific `frame-ancestors` in CSP instead of `DENY`).

---

## Rule SEC-03: Server-Side Request Forgery (SSRF) Defense [CONTEXT-DEPENDENT]
- **Rule**: When the application performs outbound HTTP requests to user-supplied URLs (webhooks, avatar imports, link previews), it MUST:
  1. Enforce `http:` and `https:` protocols exclusively.
  2. Resolve DNS prior to the request and assert that the destination IP address does NOT belong to RFC 1918 private ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), loopback (`127.0.0.1`), link-local, or cloud metadata services (`169.254.169.254`).
  3. Disable automatic HTTP redirect following.
- **Applicability**: Mandatory whenever the application fetches user-supplied URLs; not applicable if the application never initiates outbound requests to user-provided URLs.
- **Reason**: Attackers provide URLs like `http://169.254.169.254/latest/meta-data/` to force backend servers to fetch and return cloud provider IAM credentials.
- **Risk**: Full infrastructure takeover via stolen AWS/GCP administrative credentials.
- **Correct Approach**: See [SECURITY-CHECKLIST.md](../checklists/SECURITY-CHECKLIST.md#33-ssrf-defense-guardrail).
- **Exceptions**: Internal service-to-service calls using hardcoded internal URLs configured via environment variables.

---

## Rule SEC-04: Absolute Prohibition of OS Shell String Concatenation [MUST]
- **Rule**: Operating system commands MUST NOT be executed using shell-spawning functions with concatenated strings (`child_process.exec`, `os.system`). Use argument-array functions (`execFile`, `spawn`, `subprocess.run(args, shell=False)`).
- **Reason**: Passing a single concatenated string to a system shell (`/bin/sh`, `cmd.exe`) causes the shell to interpret meta-characters (`;`, `&`, `|`, `` ` ``) in user input.
- **Risk**: Arbitrary Remote Code Execution (RCE) on the server.
- **Correct Approach**:
  ```typescript
  // ❌ S1 CRITICAL: Command Injection
  exec(`zip -r archive.zip ${userSuppliedPath}`);

  // ✅ CORRECT: Argument array, no shell interpolation
  import { execFile } from 'node:child_process';
  execFile('zip', ['-r', 'archive.zip', userSuppliedPath], (err) => { ... });
  ```
- **Exceptions**: None.

---

## Rule SEC-05: Secure File Upload Processing [CONTEXT-DEPENDENT]
- **Rule**: When handling file uploads, backend services MUST:
  1. Validate the file's MIME type using "magic byte" content inspection (e.g., `file-type`), NEVER trusting the client-supplied `Content-Type` header or file extension.
  2. Enforce a strict file size limit (e.g. 5MB).
  3. Generate a completely randomized file name (`crypto.randomUUID()`) on the server.
  4. Store files outside the web server root or on object storage (Amazon S3, Google Cloud Storage) with private access policies.
- **Applicability**: Mandatory whenever applications accept user file uploads.
- **Reason**: Attackers upload malicious `.php`, `.jsp`, or `.html` files containing webshells, then browse directly to the file path to execute code on the server.
- **Risk**: Web shell execution and persistent defacement.
- **Correct Approach**:
  ```typescript
  import { fileTypeFromBuffer } from 'file-type';

  async function handleUpload(fileBuffer: Buffer) {
    const type = await fileTypeFromBuffer(fileBuffer);
    const ALLOWED_MIME_TYPES = ['image/jpeg', 'image/png', 'image/webp'];

    if (!type || !ALLOWED_MIME_TYPES.includes(type.mime)) {
      throw new SecurityError('Invalid file type detected');
    }

    const safeStorageKey = `${crypto.randomUUID()}.${type.ext}`;
    await s3Client.upload({ Bucket: 'user-uploads', Key: safeStorageKey, Body: fileBuffer });
  }
  ```
- **Exceptions**: Internal developer tooling operating strictly on local mock fixtures.
