# AI Coding Standards & Production Regulations

> **The Definitive, Practical, and Reusable Software Engineering Standard for AI Coding Agents and Human-AI Collaborative Teams.**

---

## 1. Executive Summary

As Large Language Models (LLMs) and autonomous AI coding agents take on increasingly complex software engineering responsibilities, their systemic vulnerabilities and failure modes become critical engineering risks. While AI excels at rapid boilerplate synthesis and localized problem solving, it routinely suffers from:

- **Premature over-engineering** and hallucinatory abstraction layers.
- **Silent security regressions**, unparameterized queries, and missing server-side authorization.
- **Accidental breaking changes** and loss of backward compatibility during refactoring.
- **Dependency sprawl**, importing heavy packages for trivial native operations.
- **Missing edge cases** (null states, network failures, race conditions, concurrency).
- **Accessibility and UX blind spots**, generating unsemantic `div`-soup and inaccessible controls.
- **Diff pollution**, altering unrelated files, formatting, or comments.

This repository provides an **authoritative, enforceable, and production-tested standard**. It is designed to be ingested by AI agents (Antigravity, Claude Code, Cursor, Copilot, Windsurf, Aider) as operational system prompts, referenced by human engineers during code reviews, and enforced in automated CI/CD pipelines.

---

## 2. Priority & Conflict Resolution Hierarchy

When requirements, constraints, or design goals appear to conflict during code generation, the AI agent **MUST** resolve tensions using the following strict priority order:

```text
1. Safety and Security
2. Functional Correctness
3. Explicit User / Project Requirements
4. Existing Project Constraints & Contracts
5. Data Integrity & Concurrency Safety
6. Accessibility (a11y)
7. System Reliability & Error Resilience
8. Measured Performance & Efficiency
9. Code Simplicity & Maintainability
10. Developer Convenience
```

> **Mandate**: An AI agent **MUST NEVER** weaken a higher-priority tier to satisfy a lower-priority tier. Under no circumstances may an agent disable CSRF protection, bypass authorization, or hardcode credentials to make a test pass or satisfy a user prompt. If a user requests an insecure practice, the agent **MUST** warn the user, document the risk, and implement the secure standard.

---

## 3. Normative Conformance Language

All standards in this repository use standardized requirement categories:

- **[MUST]**: Universal, mandatory requirement. Non-negotiable. Violation blocks merging.
- **[SHOULD]**: Strong recommendation with valid exceptions documented in technical rationale.
- **[MAY]**: Discretionary practice depending on project scope.
- **[CONTEXT-DEPENDENT]**: Applicability depends on architecture, technology stack, or threat model.

---

## 4. Repository Structure & Map

```text
ai-coding-standards/
│
├── README.md                           # Master repository guide and philosophy (You are here)
├── MASTER-RULES.md                     # The Master AI Coding Constitution & Protocol
│
├── core/
│   ├── GENERAL-RULES.md                # 30 Universal laws for all languages, runtimes & stacks
│   └── AI-BEHAVIOR-RULES.md            # Anti-hallucination protocols, confidence signaling, conduct
│
├── analysis/
│   └── COMMON-MISTAKES-ANALYSIS.md     # In-depth 22-category analysis of AI coding failures & remedies
│
├── checklists/
│   ├── PRE-COMPLETION-CHECKLIST.md     # Mandatory pre-commit & verification protocol for agents
│   └── SECURITY-CHECKLIST.md           # Universal vs Conditional security verification checklist
│
├── frontend/
│   ├── FRONTEND-RULES.md               # HTML5 semantics, lifecycle teardown, client form UX
│   ├── STATE-MANAGEMENT-RULES.md       # Server vs client state, URL persistence, immutability
│   ├── ACCESSIBILITY-RULES.md          # WCAG 2.2 AA, ARIA landmarks, keyboard traps, color contrast
│   ├── PERFORMANCE-RULES.md            # Core Web Vitals (LCP, INP, CLS), bundles, lazy loading
│   ├── RESPONSIVE-DESIGN-RULES.md      # Fluid layout, touch targets, container queries, viewports
│   └── SEO-RULES.md                    # Meta tags, heading hierarchy, OpenGraph, canonical URLs
│
├── backend/
│   ├── BACKEND-RULES.md                # Layered architectures, services, domain isolation, concurrency
│   ├── API-RULES.md                    # REST status codes, pagination envelopes, idempotency keys
│   ├── DATABASE-RULES.md               # SQL parameterization, indexing, ACID transactions, migrations
│   └── ERROR-HANDLING-RULES.md         # Centralized handling, operational vs programmer errors
│
├── security/
│   ├── SECURITY-RULES.md               # Defense-in-depth, zero secrets, security headers, SSRF, RCE
│   ├── AUTHENTICATION-RULES.md         # Argon2id, secure cookies, session lifecycle, MFA, rate limits
│   ├── AUTHORIZATION-RULES.md          # Server-side enforcement, RBAC/ABAC, IDOR/BOLA prevention
│   ├── DATA-PROTECTION-RULES.md        # Cryptography, PII masking, output encoding, mass assignment
│   └── DEPENDENCY-SECURITY-RULES.md    # Lockfile integrity, supply-chain vetting, CVE scanning
│
├── ui-ux/
│   ├── UI-RULES.md                     # Visual hierarchy, typography scales, 4/8pt spacing, token usage
│   ├── UX-RULES.md                     # Feedback loops, affordance, empty/loading/error states, dialogs
│   └── DESIGN-SYSTEM-RULES.md          # Component contracts, atomicity, consistent variant naming
│
├── architecture/
│   ├── ARCHITECTURE-RULES.md           # Rule of Three (anti-overengineering), inward dependency flow
│   ├── CODE-QUALITY-RULES.md           # Cognitive complexity <= 15, guard clauses, immutability
│   └── REFACTORING-RULES.md            # Safe refactoring protocol, consumer mapping, characterization
│
├── testing/
│   ├── TESTING-RULES.md                # Testing pyramid (70/20/10), determinism, boundary edge cases
│   └── QA-RULES.md                     # Severity grading (S1-S5), 11-category PR review checklist
│
├── devops/
│   ├── DEPLOYMENT-RULES.md             # Non-root Docker, health/readiness probes, graceful shutdown
│   ├── CI-CD-RULES.md                  # Automated gates, static analysis, reproducible builds, artifacting
│   ├── ENVIRONMENT-RULES.md            # 12-Factor config, startup schema validation, zero fallback secrets
│   └── MONITORING-RULES.md             # Structured JSON logging, tracing correlation IDs, RED metrics
│
└── documentation/
    └── DOCUMENTATION-RULES.md          # Architecture Decision Records (ADRs), OpenAPI, non-obvious code
```

---

## 5. How to Ingest & Use These Standards

### For AI Coding Agents
1. **Always read [MASTER-RULES.md](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-standards/MASTER-RULES.md) first** before initiating any engineering task.
2. Follow the 10 sequential operational steps: Inspect, Understand, Identify Constraints, Plan Smallest Change, Preserve Unrelated, Consider Security, Implement, Test, Review Diff, Report.
3. Review [core/GENERAL-RULES.md](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-standards/core/GENERAL-RULES.md) and [core/AI-BEHAVIOR-RULES.md](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-standards/core/AI-BEHAVIOR-RULES.md) to understand operational boundaries and anti-hallucination protocols.
4. When operating within a specific domain, inspect the corresponding specialized rulebook (e.g. [`backend/DATABASE-RULES.md`](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-standards/backend/DATABASE-RULES.md)).
5. **Execute the [PRE-COMPLETION-CHECKLIST.md](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-standards/checklists/PRE-COMPLETION-CHECKLIST.md)** and self-audit using the 13 reflection questions prior to reporting task completion.

### For Engineering Leads & Developers
- Add this directory to `.cursorrules`, `.windsurfrules`, Claude project memory, or Antigravity configuration.
- Use the 11-category review checklist and severity triage rubric in [QA-RULES.md](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-standards/testing/QA-RULES.md) for automated PR reviews.
- Integrate the [SECURITY-CHECKLIST.md](file:///c:/Users/siddh/Desktop/Projects/Rules%20and%20Regulations/ai-coding-standards/checklists/SECURITY-CHECKLIST.md) into threat modeling and staging deployment gates.
