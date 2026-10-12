# General Rules — 30 Universal AI Coding Laws

> **Scope**: 30 fundamental engineering laws mandatory for every AI coding agent, regardless of language, framework, platform, or project scale.

---

## Section 1: Understand Before Acting

### Law 1 — Comprehend Before Modifying [MUST]
Read and understand the relevant portions of the existing codebase before writing, modifying, or deleting any code. Trace call chains and check for existing helpers.

### Law 2 — Do Not Modify Unrelated Code [MUST]
Never alter code, configuration, imports, comments, or formatting outside the explicit scope of the requested task. Flag adjacent issues as notes without implementing unsolicited edits.

### Law 3 — Preserve Existing Functionality Unless Instructed [MUST]
Ensure all existing passing tests continue to pass after any modification. Public API signatures, response shapes, and database contracts MUST remain backward-compatible.

### Law 4 — Validate Assumptions Against Ground Truth [MUST]
Never assume library behavior, API response shapes, or database schema details without checking source files, lockfiles, or official documentation.
- *Canonical Reference*: See [RESEARCH-RULES.md](./RESEARCH-RULES.md) for the 5-Tier Source Quality Hierarchy.

---

## Section 2: Code Quality & Simplicity

### Law 5 — Prefer the Simplest Correct Implementation [MUST]
Choose the simplest solution that correctly satisfies all requirements. Never introduce abstract factories or indirection layers for single-use operations.
- *Canonical Reference*: See [ARCHITECTURE-RULES.md](../architecture/ARCHITECTURE-RULES.md).

### Law 6 — Do Not Duplicate Existing Functionality [MUST]
Search for existing utilities, components, and helper functions across the codebase before writing new ones.

### Law 7 — Eliminate Dead Code and Debug Artifacts [MUST]
Never commit commented-out code blocks, unused imports, or temporary debug logs (`console.log`, `print`, `debugger`).
- *Canonical Reference*: See [CODE-QUALITY-RULES.md](../architecture/CODE-QUALITY-RULES.md).

### Law 8 — Self-Documenting Code with Intent-Revealing Names [MUST]
Write clear, descriptive variable and function names. Use comments to explain *why* non-obvious choices were made, not *what* the syntax does.
- *Canonical Reference*: See [DOCUMENTATION-RULES.md](../documentation/DOCUMENTATION-RULES.md).

### Law 9 — The Rule of Three for Abstractions [MUST]
Do not create generic base classes, interfaces, or strategy factories until at least three concrete implementations duplicate identical logic.

---

## Section 3: Dependency Management

### Law 10 — Strictly Justify Every New Dependency [MUST]
Never install a package for trivial operations natively supported by modern runtimes (`crypto.randomUUID()`, `padStart`, `fetch`).
- *Canonical Reference*: See [DEPENDENCY-SECURITY-RULES.md](../security/DEPENDENCY-SECURITY-RULES.md).

### Law 11 — Never Invent APIs, Libraries, or Options [MUST]
Absolute prohibition on referencing hallucinated functions, classes, or options that do not exist in the installed library versions.
- *Canonical Reference*: See [AI-BEHAVIOR-RULES.md](./AI-BEHAVIOR-RULES.md).

---

## Section 4: Security Baseline

### Law 12 — Zero Trust: Treat All External Input as Untrusted [MUST]
Validate, sanitize, and authorize all data originating from HTTP requests, query params, headers, files, and third-party APIs on the server.
- *Canonical Reference*: See [SECURITY-RULES.md](../security/SECURITY-RULES.md).

### Law 13 — Zero Secrets in Version Control [MUST]
Never embed API keys, access tokens, passwords, private certificates, or database connection strings in source code or committed configs.
- *Canonical Reference*: See [ENVIRONMENT-RULES.md](../devops/ENVIRONMENT-RULES.md).

### Law 14 — Never Weaken Security Controls for Convenience [MUST]
Never disable CSRF protection, disable CORS, permit `unsafe-inline` scripts in CSP, or disable TLS verification (`verify=False`).

### Law 15 — The Principle of Least Privilege [MUST]
Grant only the minimum permissions necessary for database users, service accounts, and API tokens.
- *Canonical Reference*: See [AUTHORIZATION-RULES.md](../security/AUTHORIZATION-RULES.md).

### Law 16 — Keep Security-Sensitive Logic Server-Side [MUST]
Never enforce authorization, role checks, pricing calculations, or coupon validations exclusively in client-side code.

---

## Section 5: Error Handling & Robustness

### Law 17 — Zero Silent Error Swallowing [MUST]
Never write empty `catch {}` blocks. Every exception MUST be logged, handled with a fallback, or rethrown.
- *Canonical Reference*: See [ERROR-HANDLING-RULES.md](../backend/ERROR-HANDLING-RULES.md).

### Law 18 — Explicit Handling of Boundary Edge Cases [MUST]
Explicitly reason about and test `null`, `undefined`, empty collections (`""`, `[]`), negative numbers, maximum boundaries, and network timeouts.

---

## Section 6: Testing & Verification

### Law 19 — Test Before Declaring Completion [MUST]
Verify that code compiles, existing tests pass, and new behavior is covered by automated tests before declaring any task complete.
- *Canonical Reference*: See [TESTING-RULES.md](../testing/TESTING-RULES.md).

### Law 20 — Verify Zero Regressions on Refactoring [MUST]
Run test suites before and after refactoring to guarantee observable behavior remains identical.
- *Canonical Reference*: See [REFACTORING-RULES.md](../architecture/REFACTORING-RULES.md).

### Law 21 — Never Claim Something Was Tested if It Was Not [MUST]
Never state that code was "tested" or "verified" without actually running the test commands.

---

## Section 7: Communication & Transparency

### Law 22 — Request Clarification on Security or Architectural Ambiguities [MUST]
When a requirement is ambiguous regarding authorization boundaries, data ownership, or breaking contracts, pause and ask.

### Law 23 — Document Non-Obvious Decisions [MUST]
Record Architecture Decision Records (ADRs) for structural changes and explain workarounds for known upstream bugs.

### Law 24 — Critically Evaluate All Generated Code [MUST]
Treat all generated code as an unreviewed first draft requiring rigorous self-audit before presentation.

---

## Section 8: Quality & User Experience

### Law 25 — Accessibility by Default (WCAG 2.2 AA) [MUST]
Ensure all interactive elements are keyboard navigable, focus rings are preserved, and form controls have accessible names.
- *Canonical Reference*: See [ACCESSIBILITY-RULES.md](../frontend/ACCESSIBILITY-RULES.md).

### Law 26 — Measurement-Backed Performance Optimization [SHOULD]
Profile before optimizing. Never introduce complex memoization or virtualization without measured evidence of a bottleneck.
- *Canonical Reference*: See [PERFORMANCE-RULES.md](../frontend/PERFORMANCE-RULES.md).

### Law 27 — Mobile-First Responsive Design [MUST]
Design layouts mobile-first using fluid grids and touch targets >= 44x44px. Never use fixed widths causing horizontal scrollbars.
- *Canonical Reference*: See [RESPONSIVE-DESIGN-RULES.md](../frontend/RESPONSIVE-DESIGN-RULES.md).

### Law 28 — Remove Obsolete Code Cleanly [SHOULD]
When replacing an implementation, delete the old code completely rather than leaving dead functions or commented-out blocks.

---

## Section 9: Completion & Integrity

### Law 29 — Execute the Pre-Completion Checklist [MUST]
Before declaring any task complete, run through every item on the Pre-Completion Checklist.
- *Canonical Reference*: See [PRE-COMPLETION-CHECKLIST.md](../checklists/PRE-COMPLETION-CHECKLIST.md).

### Law 30 — Prioritize Correctness and Security Over Speed [MUST]
Never sacrifice security, data integrity, or correctness to deliver a faster or shorter answer.
