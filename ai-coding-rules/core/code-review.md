# Code Review Rules — AI Code Review Standard

> **Scope**: Defines the standard every AI-generated or AI-modified change MUST be reviewed against. Applies to AI self-review, peer review, and automated review tooling.

---

## 1. Severity Classification

Not all issues are equal. Reviewers MUST triage issues using this severity ladder to prevent minor stylistic preferences from being reported as blocking issues.

| Severity | Label | Definition | Action Required |
| :--- | :--- | :--- | :--- |
| **S1** | 🔴 Critical | Security vulnerability, data loss risk, authentication bypass, hardcoded secret, SQL injection, privilege escalation. | Block merge. Must fix before any delivery. |
| **S2** | 🟠 High | Functional bug that affects production users, incorrect error handling, missing server-side validation, broken accessibility. | Block merge. Must fix. |
| **S3** | 🟡 Normal | Code correctness concern, missing edge case, performance regression, dependency risk, poor naming causing confusion. | Should fix before merge. Discuss if exception is needed. |
| **S4** | 🔵 Minor | Readability improvement, better naming, minor structural simplification, missing non-critical comment. | Fix if easy; note as tech debt if not. |
| **S5** | ⚪ Optional | Style preference, alternative approach that has equivalent merit, subjective opinion. | No action required. Mention only if asked. |

> **MUST NOT** report an S5 stylistic preference as an S1 critical issue. This creates alarm fatigue and erodes trust in AI review output.

---

## 2. Review Checklist

### 2.1 — Correctness
- [ ] Does the code correctly implement the specified behavior?
- [ ] Are all inputs, outputs, and side effects correct?
- [ ] Are return values and status codes semantically correct?
- [ ] Does conditional logic cover all branches including the `else`/`default` case?
- [ ] Are boolean conditions correct (checking `===` vs `==`, `!== null` vs `!= null`)?
- [ ] Are array indices, off-by-one boundaries, and loop conditions correct?

---

### 2.2 — Security
- [ ] Are all user inputs validated server-side?
- [ ] Are all database queries parameterized (no string interpolation)?
- [ ] Is no secret, token, or credential present in the code?
- [ ] Are authorization checks present on every new route/endpoint?
- [ ] Is output encoded before rendering in HTML contexts?
- [ ] Is CSRF protection present on state-mutating endpoints?
- [ ] Is rate limiting considered on authentication and high-risk endpoints?
- [ ] Are file uploads validated for type, size, and stored outside the web root?

---

### 2.3 — Error Handling
- [ ] Are all `catch` blocks non-empty and meaningful?
- [ ] Are exceptions logged with full error context (not just `.message`)?
- [ ] Do error responses return appropriate HTTP status codes?
- [ ] Are errors surfaced to users in a safe, non-leaking way (no stack traces in production)?
- [ ] Are all async operations properly `await`ed or have explicit rejection handling?

---

### 2.4 — Edge Cases
- [ ] What happens with null, undefined, or empty inputs?
- [ ] What happens with very large inputs (pagination, file sizes, string lengths)?
- [ ] What happens with concurrent requests for the same resource?
- [ ] What happens when a third-party service is down or slow?
- [ ] What happens at the first and last item of a collection?

---

### 2.5 — Maintainability & Readability
- [ ] Are names descriptive and unambiguous?
- [ ] Is each function doing one clearly identifiable thing?
- [ ] Is there no duplicated logic that should be extracted?
- [ ] Is there no dead code, commented-out blocks, or debug artifacts?
- [ ] Can a new engineer understand this code without additional documentation?
- [ ] Are non-obvious decisions explained in comments?

---

### 2.6 — Performance
- [ ] Are there any N+1 database query patterns?
- [ ] Are large dataset operations paginated or streamed?
- [ ] Are expensive computations cached where appropriate?
- [ ] Are there any synchronous blocking operations on the main thread or event loop?
- [ ] Is there unnecessary re-rendering of UI components?
- [ ] Are images optimized and served in modern formats?

---

### 2.7 — Accessibility
- [ ] Are interactive elements (`button`, `a`, `input`) using semantic HTML?
- [ ] Do images have meaningful `alt` attributes (or `alt=""` if decorative)?
- [ ] Are form inputs associated with `<label>` elements?
- [ ] Is the focus order logical and visible?
- [ ] Are dynamic content updates announced to screen readers?
- [ ] Is color contrast at WCAG 2.2 AA (4.5:1 for normal text, 3:1 for large text)?

---

### 2.8 — Dependency Impact
- [ ] Were any new dependencies added? If so:
  - [ ] Is the dependency actively maintained?
  - [ ] Does it have known security vulnerabilities?
  - [ ] Can the need be met with an existing dependency or native API?
  - [ ] Is the bundle size impact acceptable?

---

### 2.9 — Breaking Changes & Backward Compatibility
- [ ] Were any public API signatures changed?
- [ ] Were any database schema columns modified or deleted?
- [ ] Were any exported function names or module paths changed?
- [ ] Were any environment variable names changed?
- [ ] Are there any consumers of the changed interface that were not updated?

---

### 2.10 — Unnecessary Code & Complexity
- [ ] Is all generated code actually necessary?
- [ ] Are there abstractions that serve only one implementation and add no value?
- [ ] Are there design patterns applied that are not justified by the problem complexity?
- [ ] Is there speculative/future-proofing code that is not needed today?

---

### 2.11 — Data Leakage
- [ ] Are API responses projecting only the necessary fields (not raw ORM objects)?
- [ ] Are error messages sanitized before reaching client responses?
- [ ] Are log statements free of PII, secrets, and tokens?
- [ ] Are database error messages caught before propagating to the HTTP response?

---

## 3. How to Structure Review Feedback

When providing code review output, use this format:

```
## Code Review Summary

### 🔴 Critical Issues (must fix before delivery)
1. **[Line X] SQL Injection Risk**: The query concatenates user input directly into the SQL string. Use parameterized queries.

### 🟠 High Issues (should fix before merge)
1. **[Line Y] Missing Authorization**: The `/admin/users` endpoint fetches data without verifying the requesting user has admin role.

### 🟡 Normal Issues (should address)
1. **[Line Z] Missing Error Handling**: `fetchUserData()` has no catch block. Network failures will cause unhandled promise rejections.

### 🔵 Minor Improvements
1. **[Line W] Naming**: `d` is ambiguous — consider `decodedToken` or `userPayload`.

### ⚪ Optional Suggestions
1. **[Line V] Style**: Consider extracting the validation logic into a named function for readability. Not required.
```

---

## 4. What Code Review Is NOT

**MUST NOT** use code review to:
- Impose personal stylistic preferences as correctness issues.
- Report alternative architectural approaches of equal merit as bugs.
- Demand rewrites of working, readable code to match a preferred paradigm.
- Flag every deviation from a style guide as a blocking security issue.

Code review exists to catch **correctness failures, security risks, and maintainability problems** — not to enforce aesthetic uniformity.

---

## 5. Self-Review Protocol for AI Agents

Before delivering any generated code, the AI agent **MUST** complete a self-review pass:

1. **Re-read the diff** as if you are a skeptical human reviewer seeing it for the first time.
2. **Check security items first** — scan for hardcoded secrets, injection risks, and missing authorization.
3. **Verify all imported symbols** exist in the actual dependency tree.
4. **Check error paths** — trace every `catch`, `reject`, and error return.
5. **Check for unrelated modifications** — ensure only intended lines changed.
6. **Verify tests** — confirm existing tests pass and new behavior is covered.
