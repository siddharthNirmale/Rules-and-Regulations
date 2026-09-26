# Quality Assurance (QA) & Code Review Standard

> **Authority**: Evaluation Standard for Automated CI Gates, Peer Reviews, and AI Agent Self-Audits.

---

## 1. Severity Classification Matrix

Reviewers, agents, and automated linting bots MUST triage all findings using this standardized severity ladder:

| Severity Level | Definition | Impact | Merge Action |
| :--- | :--- | :--- | :--- |
| **S1 Critical** 🔴 | Vulnerability (SQLi, IDOR, RCE, secret exposure), data corruption, auth bypass | High security threat or immediate data loss | **Block Immediately**. Non-negotiable fix required. |
| **S2 High** 🟠 | Functional breakage, broken accessibility, unhandled exception in core workflow | Severe user disruption or business logic failure | **Block Merge**. Must be resolved prior to release. |
| **S3 Normal** 🟡 | Edge-case unhandled, missing index, performance regression, inconsistent error shape | Minor defect or degraded performance | **Must Resolve**. Requires explicit approval to defer. |
| **S4 Minor** 🔵 | Minor naming confusion, code duplication under 5 lines, formatting inconsistency | Code cleanliness and readability | Fix if fast; track as debt if non-trivial. |
| **S5 Nit / Info** ⚪ | Pure aesthetic preference, equivalent alternative implementation style | Subjective stylistic opinion | **Do Not Block**. Discretionary for author. |

> **Mandate**: An AI agent or human reviewer **MUST NOT** report an S5 stylistic opinion as an S1/S2 blocking issue. This prevents alert fatigue and preserves engineering velocity.

---

## 2. Comprehensive Code Review Checklist

Reviewers and agents MUST inspect pull requests against these 11 verification categories:

### 2.1 Correctness & Logic [MUST]
- [ ] Does the code accurately implement the specified business behavior?
- [ ] Are return values, HTTP status codes, and promise resolutions semantically correct?
- [ ] Does conditional logic cover all branches, including `else` and `default` cases?
- [ ] Are equality operators strict (`===` vs `==`) and null checks explicit?
- [ ] Are array indices, off-by-one boundaries, and loop terminations verified?

### 2.2 Security & Threat Mitigation [MUST]
- [ ] Are all external inputs validated server-side using strict schemas?
- [ ] Are all database queries 100% parameterized with zero string interpolation?
- [ ] Are secrets, API keys, and bearer tokens absent from code and committed configs?
- [ ] Are authentication and authorization checks enforced on every new route?
- [ ] Is dynamic HTML sanitized with DOMPurify to eliminate XSS?
- [ ] Are file uploads restricted by magic-byte MIME type and stored outside the web root?

### 2.3 Error Handling & Resilience [MUST]
- [ ] Are all `catch` blocks non-empty with meaningful logging or recovery logic?
- [ ] Are errors logged with full context (message, stack, correlation ID)?
- [ ] Are client error responses sanitized, with zero stack traces exposed in production?
- [ ] Are all asynchronous operations properly awaited with rejection handling?

### 2.4 Edge Cases & Concurrency [MUST]
- [ ] How does the function behave with `null`, `undefined`, or empty inputs (`""`, `[]`)?
- [ ] Are collection queries safely bounded with pagination and maximum limits?
- [ ] Are concurrent data mutations guarded with transactions or row-level locks?
- [ ] What happens when downstream third-party services timeout or return 500?

### 2.5 Maintainability & Simplicity [MUST]
- [ ] Are variable, function, and component names intention-revealing?
- [ ] Does each function have a single clear responsibility (Cognitive Complexity <= 15)?
- [ ] Is there zero dead code, commented-out blocks, or temporary `console.log` statements?
- [ ] Are non-obvious business rules or upstream workarounds documented in comments?

### 2.6 Performance & Efficiency [SHOULD]
- [ ] Are there zero N+1 database queries inside loops?
- [ ] Are heavy asynchronous datasets paginated or streamed?
- [ ] Are the main thread and event loop free of synchronous blocking operations?
- [ ] Are media assets compressed, sized with aspect ratios, and lazy loaded?

### 2.7 Accessibility (a11y) [MUST]
- [ ] Are interactive elements semantic HTML (`<button>`, `<a>`, `<input>`)?
- [ ] Are form inputs associated with explicit `<label>` elements?
- [ ] Are high-contrast `:focus-visible` outlines preserved?
- [ ] Does text satisfy WCAG 2.2 AA contrast ratios (>= 4.5:1 for normal text)?

### 2.8 Dependency Impact [MUST]
- [ ] Could newly added dependencies be replaced with native standard library APIs?
- [ ] Are added packages actively maintained and free of known High/Critical CVEs?
- [ ] Are lockfiles committed and deterministic?

### 2.9 Contract Preservation & Backward Compatibility [MUST]
- [ ] Are public method signatures, REST response envelopes, and database schemas preserved?
- [ ] Were all downstream callers and consumers across the repository updated atomically?

### 2.10 Minimal Surface Area & Anti-Overengineering [MUST]
- [ ] Is this the simplest correct solution without premature generic abstractions?
- [ ] Did the change avoid introducing speculative extensibility patterns?

### 2.11 Sensitive Data & Privacy [MUST]
- [ ] Are database query results filtered to avoid leaking internal columns or passwords?
- [ ] Are logs free of customer PII, credit cards, and authentication tokens?

---

## 3. How to Structure Review Feedback

Review comments MUST be structured with severity labels and actionable remediation:

```markdown
## Code Review Summary

### 🔴 Critical Issues (S1 - Must fix before merge)
1. **[src/api/orders.ts:42] SQL Injection Hazard**:
   - Issue: Parameter `orderId` is concatenated directly into the query string.
   - Fix: Use parameter binding: `db.query('SELECT * FROM orders WHERE id = $1', [orderId])`.

### 🟠 High Issues (S2 - Must fix before release)
1. **[src/components/Checkout.tsx:115] Inaccessible Form Trigger**:
   - Issue: `<div onClick={submit}>` is not accessible via keyboard Tab/Enter.
   - Fix: Replace with native `<button type="submit">`.

### 🔵 Minor Suggestions (S4 - Non-blocking improvement)
1. **[src/utils/math.ts:18] Extract Domain Constant**:
   - Suggestion: Extract `86400000` to `const MS_PER_DAY = 86_400_000`.
```

---

## 4. Rule QA-01: Every Bug Fix MUST Include a Regression Test [MUST]
- **Rule**: When fixing a reported bug, the developer or AI agent MUST first write a failing test that reproduces the bug, and then apply the fix to make the test pass. The regression test MUST be committed in the same pull request.
- **Reason**: Any bug that occurred in production can easily recur months later during refactoring unless an automated test permanently guards that behavior.
- **Risk**: Re-introducing solved production bugs during future releases.
- **Correct Approach**:
  1. Add test reproducing the defect -> Test FAILS.
  2. Apply fix -> Test PASSES.
  3. Commit fix and test together.

---

## 5. Rule QA-02: Zero Tautological / Hollow Tests [MUST]
- **Rule**: Tests MUST assert meaningful business outcomes, not merely confirm that mocked functions were invoked. A test that passes regardless of whether the business logic is correct or broken is a "Tautological Test" and is strictly prohibited.
- **Reason**: Tautological tests inflate code coverage metrics to 95%+ while providing zero actual bug-catching capability.
- **Risk**: False sense of security and undetected production breakages.

---

## 6. Rule QA-03: Mutation Resistance Verification [SHOULD]
- **Rule**: To verify test quality, an engineer or AI agent SHOULD ask: *"If I invert this `>` to a `<`, or change this `+` to a `-`, will at least one test fail?"* If the tests still pass when the business logic is intentionally corrupted, the test suite is inadequate.
- **Reason**: High line coverage does not equal high assertion coverage. Mutation testing ensures that assertions actually validate data transformations.
