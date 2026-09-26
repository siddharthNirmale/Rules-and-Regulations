# Data Protection & Cryptography Rules

> **Scope**: Encryption at rest and in transit, PII handling, input sanitization, output encoding, and mass-assignment defense.

---

## Rule DP-01: Protect Personally Identifiable Information (PII) & Automated Redaction [MUST]
- **Rule**: Personally Identifiable Information (PII)—including Social Security Numbers, credit card numbers, home addresses, phone numbers, and full names—MUST NOT be written to application logs, crash reports, analytics streams, or unencrypted storage.
- **Canonical Reference**: For automated logging redaction configuration, see [MONITORING-RULES.md](../devops/MONITORING-RULES.md#rule-mon-02-mandatory-automated-redaction-of-sensitive-data-must).
- **Reason**: Log files are widely distributed among developers, log aggregation vendors, and monitoring dashboards. Unredacted PII in logs violates GDPR, HIPAA, and PCI-DSS regulations.
- **Risk**: Massive regulatory fines and reputational collapse.
- **Exceptions**: Explicit audit trails stored in dedicated, encrypted, compliance-certified audit databases with restricted access.

---

## Rule DP-02: Authenticated Encryption at Rest (AES-256-GCM) [CONTEXT-DEPENDENT]
- **Rule**: When storing sensitive data at rest (such as API keys, OAuth refresh tokens, or personal identifiers), systems MUST use modern authenticated encryption (such as **AES-256-GCM** or **ChaCha20-Poly1305**). Unauthenticated cipher modes like AES-CBC without HMAC or AES-ECB MUST NEVER be used.
- **Applicability**: Mandatory whenever storing secrets, external API keys, or sensitive customer tokens in application databases.
- **Reason**: Authenticated encryption provides both confidentiality (keeping data secret) and integrity (ensuring the ciphertext has not been tampered with or modified).
- **Risk**: Padding oracle attacks and bit-flipping data corruption attacks.
- **Correct Approach**:
  ```typescript
  import crypto from 'node:crypto';

  const ALGORITHM = 'aes-256-gcm';
  const IV_LENGTH = 12; // 96 bits recommended for GCM

  export function encrypt(plainText: string, key: Buffer): string {
    const iv = crypto.randomBytes(IV_LENGTH);
    const cipher = crypto.createCipheriv(ALGORITHM, key, iv);
    
    let encrypted = cipher.update(plainText, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    const authTag = cipher.getAuthTag().toString('hex');

    return `${iv.toString('hex')}:${authTag}:${encrypted}`;
  }
  ```
- **Exceptions**: Data protected by full-disk or cloud-provider transparent database encryption (TDE), provided cryptographic keys are managed securely in KMS.

---

## Rule DP-03: Context-Aware Output Encoding (XSS Defense) [MUST]
- **Rule**: Before untrusted user-supplied data is rendered into HTML, attributes, JavaScript blocks, or URLs, it MUST be encoded appropriately for that specific output context.
- **Reason**: Modern browsers execute scripts when unescaped HTML characters (`<`, `>`, `"`, `'`, `&`) appear inside dynamic templates.
- **Risk**: Stored and Reflected Cross-Site Scripting (XSS).
- **Correct Approach**:
  - Prefer modern frameworks (React, Vue, Svelte) that automatically perform context-aware encoding.
  - When rendering raw HTML is strictly necessary, sanitize with **DOMPurify** before rendering.
  ```typescript
  // ❌ S1 REJECTION: Unsanitized HTML rendering
  <div dangerouslySetInnerHTML={{ __html: userBio }} />

  // ✅ CORRECT: Sanitized via DOMPurify
  import DOMPurify from 'dompurify';
  <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userBio) }} />
  ```
- **Exceptions**: None.

---

## Rule DP-04: Strict Mass Assignment Protection [MUST]
- **Rule**: API endpoints MUST NOT pass raw request bodies directly to database creation or update methods. Every incoming payload MUST be filtered through an explicit schema or allowlist.
- **Reason**: Attackers add unexpected JSON fields like `{ "role": "ADMIN", "isVerified": true, "balance": 10000 }` to regular profile update payloads.
- **Risk**: Unauthorized privilege escalation and account hijacking.
- **Correct Approach**:
  ```typescript
  // ❌ S1 CRITICAL: Mass assignment
  await prisma.user.update({ where: { id }, data: req.body });

  // ✅ CORRECT: Explicit property picking or schema parsing
  const updateData = {
    bio: req.body.bio,
    avatarUrl: req.body.avatarUrl
  };
  await prisma.user.update({ where: { id }, data: updateData });
  ```
- **Exceptions**: None.
