# Documentation & Architecture Decision Rules

> **Scope**: Architecture Decision Records (ADRs), code commentary ethics, API specifications, and repository documentation.

---

## Rule DOC-01: Document the "Why", Not the "What"
- **Rule**: Code comments MUST NOT explain what the syntax is doing if the syntax is self-evident. Comments MUST be reserved for explaining *why* an unusual technical choice was made, documenting non-obvious business rules, citing regulatory statutes, or warning about subtle edge-case workarounds.
- **Reason**: Tautological comments (`// increment i: i++;`) clutter files, provide zero value, and quickly drift out of sync with code changes.
- **Risk**: Cognitive noise, developer fatigue, and misleading outdated comments.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Tautological comment
  // Get the customer by id
  function getCustomer(id: string) {
    // Return customer from database
    return db.customers.find(id);
  }

  // ✅ CORRECT: Explains non-obvious business constraint & external vendor limitation
  /**
   * Stripe limits idempotency key retention to 24 hours. We enforce a 20-hour
   * internal TTL to ensure keys never expire mid-retry on distributed retries.
   * See Stripe API Docs: https://stripe.com/docs/api/idempotent_requests
   */
  const IDEMPOTENCY_TTL_SECONDS = 20 * 60 * 60;
  ```
- **Exceptions**: JSDoc/TSDoc on public exported library interfaces describing parameter types and return contracts for external consumers.

---

## Rule DOC-02: Mandatory Architecture Decision Records (ADRs)
- **Rule**: Significant architectural choices (e.g. adopting a new database, replacing an auth framework, choosing between REST vs GraphQL, introducing a queue system) MUST be documented in a lightweight Architecture Decision Record (ADR) stored in `docs/adr/`.
- **Reason**: When teams or AI agents do not know *why* an architectural decision was made, future developers often reverse the decision without understanding the historical constraints that motivated it, repeating past mistakes.
- **Risk**: Architectural churn and repeated failures.
- **Standard ADR Template**:
  ```markdown
  # ADR 004: Adoption of PostgreSQL Row-Level Security for Multi-Tenancy

  ## Status
  Accepted (2024-03-15)

  ## Context
  Our B2B SaaS platform requires strict tenant isolation. Relying on application-level
  `WHERE tenant_id = ?` filters has previously resulted in a staging data leak when a junior
  developer missed the filter on an export query.

  ## Decision
  We will enforce Row-Level Security (RLS) directly in PostgreSQL. All database connections
  will execute `SET app.current_tenant_id = ?` upon checkout from the connection pool.

  ## Consequences
  - Positive: Zero possibility of accidental cross-tenant data leaks in application queries.
  - Positive: Eliminates boilerplate tenant filtering across 100+ repository methods.
  - Negative: Direct table queries run by DBA or migration scripts must explicitly bypass RLS.
  ```
- **Exceptions**: Routine feature work and standard bug fixes that fit within existing architectural patterns.

---

## Rule DOC-03: Keep API Contracts (OpenAPI / Types) in Lockstep
- **Rule**: When adding or modifying backend API routes, the corresponding OpenAPI specification, Swagger documentation, or exported TypeScript contract types MUST be updated in the same pull request.
- **Reason**: Discrepancies between documentation and runtime API implementations cause frontend client failures, broken automated SDK generation, and developer confusion.
- **Risk**: Broken third-party integrations and mismatched payload schemas.
- **Correct Approach**:
  - Prefer schema-first or code-first automatic generation (e.g. `zod-to-openapi`, NestJS Swagger decorators, FastAPI automatic OpenAPI).
  - Include contract validation tests in CI asserting that runtime responses match the documented schema.
- **Exceptions**: Internal private RPC endpoints that are strictly un-exported to third parties and covered by end-to-end integration tests.

---

## Rule DOC-04: Up-to-Date Repository README Protocol
- **Rule**: Every repository MUST maintain an up-to-date `README.md` containing:
  1. **System Overview**: High-level purpose and core tech stack.
  2. **Prerequisites**: Required runtime versions (Node, Python, Docker).
  3. **Local Setup & Seeding**: Exact step-by-step commands to clone, configure `.env`, install dependencies, and run migrations.
  4. **Running Locally**: Command to start development servers (`npm run dev`).
  5. **Running Tests & Linters**: Commands to execute full verification (`npm test`, `npm run lint`).
- **Reason**: Developers and autonomous AI agents need an immediate, verified onboarding guide to build and test the project without guessing commands.
- **Risk**: Onboarding friction, lost productivity, and broken CI pipelines.
- **Exceptions**: None.
