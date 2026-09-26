# Environment Configuration & 12-Factor Rules

> **Standard**: 12-Factor App Configuration, Strict Schema Bootstrapping, and Environment Isolation.

---

## The 12-Factor Config Principle
An application's code repository MUST be completely agnostic of the deployment target. The exact same built binary or container image MUST run identically in Development, Staging, and Production by varying only external environment variables.

---

## Rule ENV-01: Centralized, Schema-Validated Startup Configuration
- **Rule**: All environment variables MUST be validated against a strict schema (e.g. Zod, Pydantic, Envalid) during the application's boot sequence. If any required variable is missing, malformed, or fails boundary checks, the application MUST log a fatal error and terminate immediately (`process.exit(1)`).
- **Reason**: Accessing `process.env` ad-hoc inside nested functions allows an application to boot successfully with missing configuration, only to crash hours later when a customer triggers the unconfigured code path.
- **Risk**: Mid-transaction crashes and unexpected downtime.
- **Correct Approach**:
  ```typescript
  // src/config/env.ts
  import { z } from 'zod';

  const EnvSchema = z.object({
    NODE_ENV: z.enum(['development', 'test', 'production']),
    PORT: z.coerce.number().default(3000),
    DATABASE_URL: z.string().url(),
    REDIS_URL: z.string().url().optional(),
    JWT_SECRET: z.string().min(32, 'JWT_SECRET must be at least 32 characters'),
    SENTRY_DSN: z.string().url().optional()
  });

  const parseResult = EnvSchema.safeParse(process.env);
  if (!parseResult.success) {
    console.error('❌ FATAL: Invalid configuration environment variables:');
    console.error(parseResult.error.format());
    process.exit(1); // Fail immediately before binding ports
  }

  export const config = parseResult.data;
  ```
- **Exceptions**: None. All production services must fail-fast on configuration errors.

---

## Rule ENV-02: Zero Insecure Fallback Defaults for Secrets
- **Rule**: Security-critical configurations (session secrets, database passwords, encryption keys, private tokens) MUST NOT declare fallback default values (e.g., `process.env.JWT_SECRET || 'secret123'`). They MUST be marked strictly required.
- **Reason**: Developers add convenient fallback defaults for local testing, but when the environment variable is accidentally omitted in staging or production, the application silently boots using the public, well-known fallback secret.
- **Risk**: Critical vulnerability allowing attackers to forge auth tokens or decrypt data using the public default string.
- **Correct Approach**:
  ```typescript
  // ❌ S1 CRITICAL: Insecure fallback
  const secret = process.env.SESSION_SECRET || 'dev_secret_fallback';

  // ✅ CORRECT: Schema demands explicit non-empty string in all environments
  SESSION_SECRET: z.string().min(32)
  ```
- **Exceptions**: Non-sensitive operational toggles (e.g., `PORT: default(3000)`, `LOG_LEVEL: default('info')`).

---

## Rule ENV-03: Complete Physical Segregation of Environments
- **Rule**: Staging and Production environments MUST NOT share database instances, Redis caches, cloud storage buckets, or cryptographic keys. Staging data MUST NOT contain live customer PII or production user credentials.
- **Reason**: Sharing persistence layers across environments guarantees that a test run or staging migration will accidentally overwrite, corrupt, or leak live production customer data.
- **Risk**: Production data destruction and massive regulatory data breaches.
- **Correct Approach**:
  - Distinct AWS/GCP accounts or isolated VPCs per environment.
  - Distinct database clusters with separate IAM credentials.
  - Staging databases populated strictly with synthetic or anonymized seed data.
- **Exceptions**: None.

---

## Rule ENV-04: Mandatory Maintenance of `.env.example`
- **Rule**: Repositories MUST maintain an up-to-date `.env.example` file that lists every single environment variable required by the application, annotated with comments explaining its format and purpose. The `.env` file itself MUST be listed in `.gitignore`.
- **Reason**: Onboarding new developers and deploying new CI/CD runners becomes a frustrating guessing game when required environment variables are undocumented.
- **Risk**: Deployment failures and accidental commitment of live `.env` files containing secrets.
- **Correct Approach**:
  ```ini
  # .env.example (Safe to commit)
  NODE_ENV=development
  PORT=3000
  # Connection string for primary PostgreSQL instance
  DATABASE_URL=postgresql://user:password@localhost:5432/dbname
  # Minimum 32-character secret key for JWT session signing
  JWT_SECRET=replace_with_secure_random_32_character_string
  ```
- **Exceptions**: None.
