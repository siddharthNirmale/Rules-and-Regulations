# AI Coding Rules & Regulations: Master System

> **The definitive production standard for AI coding agents and human-AI collaborative software engineering.**

---

## 1. Purpose & Vision

The **AI Coding Rules & Regulations** system establishes an uncompromising, production-grade standard for AI coding agents designing, writing, reviewing, refactoring, and maintaining software. 

While Large Language Models (LLMs) excel at rapid code synthesis, they are susceptible to characteristic failure modes: over-engineering, hallucinated APIs, broken edge cases, regression during refactoring, security omissions, accessibility failures, and dependency sprawl.

This repository serves as:
1. **A Persistent Instruction Set**: A drop-in ruleset for IDE agents (such as Antigravity, Claude Code, Cursor, Copilot Workspace, Windsurf, Aider) and CI/CD automated review bots.
2. **An Architectural Standard**: A reference manual for development teams establishing human-AI collaborative standards.
3. **A Quality Gate**: Comprehensive checklists and evaluation frameworks that must be satisfied before any AI-generated code merges to production.

---

## 2. Rule Hierarchy & Conflict Resolution

When requirements or guidelines appear to conflict during code generation, the AI agent **MUST** resolve tensions using the following strict precedence ladder:

```
┌────────────────────────────────────────────────────────┐
│ 1. SECURITY & DATA INTEGRITY                          │
│    (Zero-trust, OWASP ASVS, secret safety, no bypasses)│
├────────────────────────────────────────────────────────┤
│ 2. FUNCTIONAL CORRECTNESS & ROBUSTNESS                 │
│    (Correct logic, edge-case safety, graceful failure) │
├────────────────────────────────────────────────────────┤
│ 3. PRESERVATION OF EXISTING CONTRACTS                  │
│    (No unintended breaking changes, backward-compat)   │
├────────────────────────────────────────────────────────┤
│ 4. EXPLICIT PROJECT SPECIFICATIONS                     │
│    (User prompt, project PRD, domain constraints)      │
├────────────────────────────────────────────────────────┤
│ 5. ACCESSIBILITY & INCLUSIVITY                         │
│    (WCAG 2.2 AA compliance, semantic markup)           │
├────────────────────────────────────────────────────────┤
│ 6. SIMPLICITY & MAINTAINABILITY                        │
│    (Least code required, readability, standard idioms) │
├────────────────────────────────────────────────────────┤
│ 7. PERFORMANCE & EFFICIENCY                            │
│    (Measurement-backed optimization, no premature bloat│
├────────────────────────────────────────────────────────┤
│ 8. AESTHETICS & MICROINTERACTIONS                      │
│    (Polish, visual hierarchy, purposeful animations)   │
└────────────────────────────────────────────────────────┘
```

> **Mandate**: An AI agent **MUST NEVER** weaken Security or Correctness to satisfy Performance, Simplicity, or an ambiguous user request. If an explicit user instruction requests an insecure practice (e.g., *"disable CSRF protection for testing"* or *"hardcode the API key"*), the agent **MUST** warn the user, document the hazard, and propose the secure standard.

---

## 3. Normative Vocabulary (RFC 2119 Standards)

All files within this ruleset utilize standardized requirement keywords:

* **MUST / SHALL / REQUIRED**: Absolute requirement. Violation constitutes an immediate failure and pull-request rejection.
* **MUST NOT / SHALL NOT**: Absolute prohibition.
* **SHOULD / RECOMMENDED**: Strong recommendation. Valid exceptions may exist in specific contexts, but must be justified and documented.
* **SHOULD NOT / NOT RECOMMENDED**: Practice that is discouraged unless proven necessary with clear rationale.
* **MAY / OPTIONAL**: Truly discretionary behavior.

---

## 4. Repository & Knowledge Organization

This rule system is divided into modular, single-responsibility domains:

```text
ai-coding-rules/
│
├── README.md                           # Master system guide & precedence rules
│
├── core/
│   ├── general-rules.md                # 30 Universal laws for all languages & stacks
│   ├── ai-coding-behavior.md           # Behavioral protocol, anti-hallucination, verification
│   ├── common-mistakes.md              # 24 Canonical AI coding failures, roots & preventions
│   ├── refactoring-rules.md            # Safe refactoring protocols (preserve, verify, minimize)
│   └── code-review.md                  # PR review rubric, severity triage (Critical to Nit)
│
├── frontend/
│   ├── frontend-rules.md               # HTML5 semantics, JS/TS, state, forms, client security
│   ├── react-rules.md                  # React 18/19 patterns, hooks, lifecycle, server components
│   ├── css-tailwind-rules.md           # Modern CSS, design tokens, Tailwind architecture
│   └── accessibility.md                # WCAG 2.2 AA standards, ARIA, keyboard navigation
│
├── backend/
│   ├── backend-rules.md                # Layered architecture, controllers, services, repositories
│   ├── api-rules.md                    # RESTful design, idempotency, versioning, contract schemas
│   ├── database-rules.md               # Parameterization, indexing, ACID transactions, migrations
│   └── error-handling.md               # Centralized errors, safe logging, operational vs programmer
│
├── ui-ux/
│   ├── ui-rules.md                     # Visual hierarchy, typography, 4/8px spacing, color systems
│   ├── ux-rules.md                     # Affordance, feedback, zero-data/empty/error states
│   ├── responsive-design.md            # Fluid layouts, mobile-first, touch targets, container queries
│   └── animation-rules.md              # Purposeful motion, 150-300ms timings, prefers-reduced-motion
│
├── security/
│   ├── security-rules.md               # Defense-in-depth, zero-trust architecture, OWASP ASVS
│   ├── web-security.md                 # XSS, CSRF, CORS, CSP, Clickjacking, Security Headers
│   ├── authentication.md               # Passwords (Argon2id), session management, MFA, JWTs
│   ├── authorization.md                # Server-side enforcement, RBAC/ABAC, IDOR/BOLA prevention
│   ├── data-security.md                # Input sanitization, encryption at rest/in transit, PII masking
│   ├── api-security.md                 # Rate limiting, throttling, payload limits, SSRF prevention
│   └── dependency-security.md          # Supply-chain hygiene, lockfile integrity, audit workflows
│
├── testing/
│   ├── testing-rules.md                # Testing pyramid, unit/integration/e2e/security testing
│   └── test-review.md                  # Behavioral verification vs test-count inflation
│
├── performance/
│   └── performance-rules.md            # Bundle budgets, Core Web Vitals (LCP, INP, CLS), query tuning
│
├── architecture/
│   └── architecture-rules.md           # Modularity, separation of concerns, DRY without over-abstracting
│
├── documentation/
│   └── documentation-rules.md          # Architecture Decision Records (ADRs), OpenAPI, non-obvious code
│
└── checklists/
    ├── pre-deployment.md               # Production go-live gating checklist
    ├── security-checklist.md           # Comprehensive vulnerability verification checklist
    └── final-ai-review.md              # Final self-audit checklist before task sign-off
```

---

## 5. Universal vs. Technology-Specific Rules

To prevent confusion, understand which rules apply unconditionally and which adapt to the stack:

| Category | Scope | Examples |
| :--- | :--- | :--- |
| **Universal Rules** | All languages, frameworks, scripts, infrastructure-as-code | Parameterized database queries, zero secrets in source code, never modifying unrelated files, verifying before completing, strict input validation, least-privilege permissions. |
| **Domain Rules** | Frontend, Backend, UI/UX, DevOps | Web accessibility (WCAG), REST status code usage, responsive breakpoints, database index design. |
| **Technology-Specific** | React, Next.js, Node.js, Tailwind, Postgres, Docker | React `useEffect` dependency array correctness, Tailwind utility composition, Postgres JSONB indexing. |

---

## 6. How AI Coding Agents Must Apply These Rules

### Scenario A: Green-Field Feature Implementation
1. **Review Architecture**: Check [`architecture/architecture-rules.md`](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-rules/architecture/architecture-rules.md) and stack standards before proposing file layouts.
2. **Draft Core Logic First**: Implement standard, minimal, dependency-free logic.
3. **Enforce Security By Default**: Apply validation schemas ([`backend/api-rules.md`](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-rules/backend/api-rules.md)), authorization checks ([`security/authorization.md`](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-rules/security/authorization.md)), and parameterization immediately.
4. **Build Resilient UI/UX**: Provide loading, empty, error, and success states alongside accessible markup.
5. **Verify**: Run unit and integration tests covering positive and negative edge cases.

### Scenario B: Refactoring Existing Code
1. **Analyze First**: Read and comprehend the existing implementation and test coverage.
2. **Characterize Current Behavior**: Ensure existing unit/integration tests pass before modifying any code.
3. **Make Atomic Changes**: Refactor one isolated concern at a time without altering external interfaces unless requested.
4. **Preserve Contracts**: Keep public method signatures, API response envelopes, and database columns intact.
5. **Never Touch Unrelated Code**: Adhere strictly to the bounding box of the requested task.

### Scenario C: Bug Fixing & Troubleshooting
1. **Reproduce First**: Formulate an exact mental or executable reproduction of the failure mode.
2. **Identify Root Cause**: Avoid applying speculative "band-aids" (e.g., adding arbitrary `try/catch` or `setTimeout`).
3. **Implement Minimal Fix**: Fix the root cause cleanly at the correct layer (e.g., sanitize at ingest, not in presentation).
4. **Add Regression Test**: Ensure a test asserts the bug cannot silently recur.

---

## 7. How to Perform the Final Review

Before declaring any coding task complete or presenting diffs to human reviewers, the AI agent **MUST** run the [Final Self-Review Protocol](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-rules/checklists/final-ai-review.md):

1. **Diff Audit**: Inspect every modified line against git status. Are there any unintended white-space changes, commented-out dead code, or unrelated file modifications?
2. **Hallucination Check**: Were any imported packages, functions, or CSS properties invented or assumed?
3. **Security Audit**: Are all inputs validated? Are secrets absent? Is authorization enforced on every new route?
4. **Accessibility & Usability Check**: Can every UI element be used with only a keyboard? Are color contrasts sufficient?
5. **Build & Test Execution**: Did the build, linter, typecheck, and test suite execute with zero errors?

---

## 8. Continuous Maintenance & Evolution

This system is a living standard. It must be updated when:
* **New Security Advisories Emerge**: CVEs or updated OWASP guidance demand stricter mitigation.
* **Framework Paradigms Advance**: Language specifications (e.g., ECMAScript) or framework versions deprecate legacy patterns.
* **Recurring Agent Blindspots are Discovered**: When an AI agent repeatedly makes a novel mistake, add the pattern to [`core/common-mistakes.md`](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-rules/core/common-mistakes.md) along with its enforceable prevention rule.
