# General Rules — Universal AI Coding Laws

> **Scope**: These 30 rules are **MANDATORY** for every AI coding agent, regardless of language, framework, platform, or project type. No exception is valid without explicit, documented justification from the project owner.

---

## Section 1: Understand Before Acting

### Rule 1 — Comprehend the Codebase Before Modifying It
**MUST** read and understand the relevant portions of the existing codebase before writing, modifying, or deleting any code.

- Identify the module's purpose, entry points, data contracts, and dependencies.
- Trace call chains for functions you intend to modify.
- Confirm whether similar utilities, helpers, or abstractions already exist.

> **Why**: The most common source of AI-introduced bugs is blind modification of code whose side-effects or behavioral contracts were not understood.

---

### Rule 2 — Do Not Modify Unrelated Code
**MUST NOT** alter code, configuration, imports, comments, or formatting outside the explicit scope of the requested task.

- If you notice an adjacent bug or improvement, flag it as a note **without implementing it**.
- Whitespace-only changes to unrelated lines are prohibited.

> **Why**: Unrelated changes introduce unexpected regressions and pollute code reviews, making it impossible to reason about the actual change.

---

### Rule 3 — Preserve Existing Functionality Unless Explicitly Instructed
**MUST** ensure that all existing, passing tests continue to pass after any modification.

- Behavior changes MUST be deliberate, documented, and sanctioned by the task specification.
- If refactoring is required, the public interface MUST remain unchanged unless the task explicitly demands otherwise.

---

### Rule 4 — Validate Assumptions Before Implementing
**MUST NOT** assume library behavior, API response shapes, framework quirks, or database schema details that were not verified against actual source files, official documentation, or project code.

- If uncertain, surface the assumption to the user before proceeding.
- Unverified assumptions MUST be marked with `// ASSUMPTION: ...` comments if they cannot be avoided.

---

## Section 2: Code Quality & Simplicity

### Rule 5 — Prefer the Simplest Correct Implementation
**MUST** choose the simplest implementation that correctly satisfies all requirements.

- Never add abstraction layers, generics, or configurability that is not immediately needed.
- "Clever" code is not clever if a future engineer cannot read it in 30 seconds.

**Acceptable**: Straightforward loop or conditional.
**Unacceptable**: Custom visitor pattern or macro system for a 3-step transformation.

---

### Rule 6 — Do Not Duplicate Existing Functionality
**MUST** search for existing utilities, services, helpers, and components that already satisfy the need before writing new code.

- Use existing validation schemas, date formatters, API clients, error handlers, and logging utilities.
- Duplication creates maintenance debt and desynchronized behavior over time.

---

### Rule 7 — Remove Unnecessary Code
**MUST NOT** leave dead code, unused imports, commented-out blocks, or debug artifacts (`console.log`, `print`, `debugger`) in committed code.

- Deletions are improvements.
- Code that serves no current function creates cognitive load and confusion.

---

### Rule 8 — Write Code That Another Developer Can Understand
**MUST** write code as if a competent engineer will read it without additional context.

- Variable and function names MUST be descriptive and unambiguous.
- Abbreviations MUST follow established project conventions, not ad hoc intuition.
- Complex logic MUST include a comment explaining *why*, not *what*.

**Good**:
```js
// Expire sessions after 15 minutes of inactivity per compliance policy RQ-142
const SESSION_IDLE_TIMEOUT_MS = 15 * 60 * 1000;
```
**Bad**:
```js
const t = 900000; // timeout
```

---

### Rule 9 — Do Not Create Abstractions Without a Clear Benefit
**MUST NOT** create interfaces, base classes, factory functions, plugin systems, or inversion-of-control containers unless there are at least two concrete use cases that immediately justify the abstraction.

- Premature abstraction is a form of unnecessary complexity.
- The cost of wrong abstraction is higher than the cost of repetition.

> **Why**: AI agents frequently introduce speculative generalization. This creates entanglement that is harder to remove than straightforward duplicate code.

---

## Section 3: Dependency Management

### Rule 10 — Do Not Add Dependencies Without Justification
**MUST** justify every new dependency addition against these criteria:

1. Does the project already include an equivalent package?
2. Can this be implemented in fewer than ~50 lines of standard-library code?
3. Is the package actively maintained with recent commits?
4. Does it introduce known security vulnerabilities?
5. Does it significantly increase the artifact/bundle size?

If any question raises a red flag, **do not add the dependency**.

---

### Rule 11 — Never Invent APIs, Libraries, or Framework Behavior
**MUST NOT** reference, import, or call any function, class, configuration option, or library that has not been confirmed to exist in the project's actual dependency tree or official documentation.

This is an absolute prohibition. The consequence of invented APIs is runtime crashes in production.

**Banned patterns**:
```js
// WRONG — invented hook that does not exist in React
import { useOptimisticMutation } from 'react'; 

// WRONG — invented Tailwind utility
className="text-semantic-primary-alt"
```

---

## Section 4: Security

### Rule 12 — Treat All External Input as Untrusted
**MUST** validate, sanitize, and authorize all input received from any source external to the current trust boundary: HTTP requests, URL parameters, form fields, file contents, database results, environment variables, third-party APIs, IPC messages.

The rule applies regardless of whether the source is "internal" — insider threat and supply-chain compromise are real attack vectors.

---

### Rule 13 — Never Hardcode Secrets, Credentials, or API Keys
**MUST NOT** embed secrets, tokens, passwords, connection strings, private keys, or certificates in source code, configuration files committed to version control, or client-side bundles.

- All secrets MUST be loaded from environment variables or a secrets manager at runtime.
- If a secret is discovered in generated code, the agent MUST immediately flag it and remove it.

---

### Rule 14 — Never Weaken Security to Simplify Implementation
**MUST NOT** disable, bypass, reduce, or remove security controls because they are inconvenient, cause test failures, or conflict with a quicker implementation path.

**Banned patterns**:
```python
# WRONG
verify=False  # in requests.get() — disables TLS verification

# WRONG
csrf_exempt  # applied broadly without specific, documented justification

# WRONG
app.use(cors({ origin: '*' }))  # wildcard CORS on an authenticated API
```

---

### Rule 15 — Follow the Principle of Least Privilege
**MUST** request, grant, and operate with the minimum permissions necessary.

- Database credentials for a read-only report endpoint MUST NOT have write access.
- Service accounts MUST NOT have administrator-level roles unless actively required.
- Frontend tokens MUST NOT carry scopes beyond what the client's features need.

---

### Rule 16 — Keep Security-Sensitive Logic Server-Side
**MUST NOT** implement access control, pricing calculation, permission evaluation, coupon validation, or any business-critical logic on the client.

Client-side code is fully observable and mutable by end users. Any security control placed exclusively in the browser or mobile client MUST be treated as absent.

---

## Section 5: Error Handling & Robustness

### Rule 17 — Handle Errors Explicitly
**MUST** handle error conditions with specific, meaningful recovery logic.

- Do not use empty `catch {}` blocks.
- Do not silently swallow exceptions without at minimum logging the error.
- Distinguish between recoverable operational errors (network timeout) and programmer errors (null dereference).

**Good**:
```ts
try {
  await sendEmail(user.email, template);
} catch (err) {
  logger.error({ err, userId: user.id }, 'Email delivery failed');
  // Allow user to continue — email is non-blocking
}
```
**Bad**:
```ts
try {
  await sendEmail(user.email, template);
} catch (e) {} // swallowed
```

---

### Rule 18 — Consider and Handle Edge Cases
**MUST** explicitly consider and handle common edge cases for every non-trivial function:

- Empty collections, zero-length strings, null/undefined inputs.
- Off-by-one boundaries (first/last page, empty dataset, maximum value).
- Race conditions when multiple concurrent users or requests are plausible.
- Clock skew, timezone differences, DST transitions for date/time logic.

---

## Section 6: Testing & Verification

### Rule 19 — Test Before Considering a Task Complete
**MUST** verify that:

1. The implemented code compiles/parses without errors.
2. Existing tests continue to pass.
3. New logic is covered by at least one meaningful test.
4. Manual smoke-testing confirms the visible behavior matches the specification.

An AI agent **MUST NOT** declare a task complete if it has only generated code without executing verification.

---

### Rule 20 — Verify That Refactoring Does Not Break Existing Behavior
**MUST** run the full test suite before and after any refactoring task. Any newly failing test is a blocker that must be resolved before delivering the change.

---

### Rule 21 — Never Claim Something Was Tested if It Was Not
**MUST NOT** report that code was "tested," "verified," or "confirmed working" without having actually executed it or reviewed its test coverage.

> **Why**: False confidence in AI-reported test results is a documented failure mode that leads to production incidents.

---

## Section 7: Communication & Transparency

### Rule 22 — Ask for Clarification When Ambiguity Affects Correctness or Security
**MUST** pause and request clarification when a requirement is ambiguous in a way that would lead to materially different implementations — especially when the ambiguity involves security logic, data handling, or architectural contracts.

Making an incorrect assumption about authorization rules is more harmful than asking one clarifying question.

---

### Rule 23 — Document Important Non-Obvious Decisions
**MUST** leave a brief comment or inline documentation when:

- A non-obvious design choice was made (e.g., idempotency key strategy).
- A security control is intentionally applied in an unusual location.
- A workaround for a known library bug or platform quirk was necessary.

---

### Rule 24 — Do Not Blindly Trust Generated Code
**MUST** treat all generated code — including your own output — as an unreviewed first draft that requires critical evaluation.

- Check generated regex patterns for correctness and catastrophic backtracking.
- Verify generated SQL for injection risk and correctness of query semantics.
- Verify generated HTML for semantic accuracy and ARIA attribute correctness.

---

## Section 8: Accessibility, Performance & Maintainability

### Rule 25 — Consider Accessibility and Responsive Behavior
**MUST** ensure that every UI component:

- Is navigable by keyboard alone.
- Has sufficient color contrast (WCAG 2.2 AA minimum: 4.5:1 for normal text).
- Provides appropriate ARIA roles and labels for screen readers.
- Renders functionally on both mobile and desktop viewport widths.

---

### Rule 26 — Consider Performance Where It Materially Affects User Experience
**SHOULD** evaluate whether a proposed implementation introduces measurable performance regressions — particularly for:

- Database queries adding N+1 patterns.
- Synchronous blocking operations on the main thread or event loop.
- Components that re-render on every keystroke without memoization.

**MUST NOT** prematurely optimize code in the absence of evidence of a performance problem.

---

## Section 9: Dependencies & Maintenance

### Rule 27 — Keep Dependencies Current Where Practical
**SHOULD** use dependency versions that are not end-of-life or known to carry unpatched vulnerabilities.

- Run `npm audit`, `pip audit`, `cargo audit`, or equivalent before finalizing dependency selections.
- Prefer LTS releases of runtimes and frameworks for production applications.

---

### Rule 28 — Remove Obsolete Code When Appropriate
**SHOULD** remove code that has been superseded by a new implementation during a refactoring or feature change, rather than leaving it as dead code or comment blocks.

---

## Section 10: Final Protocol

### Rule 29 — Perform a Final Review Before Sign-Off
**MUST** before completing any task, run a final mental (or automated) sweep across:

- ✅ Functionality correct and complete
- ✅ Security controls applied
- ✅ No unrelated files modified
- ✅ No secrets in the diff
- ✅ Tests are passing
- ✅ Accessibility is considered
- ✅ Error paths are handled

See [`checklists/final-ai-review.md`](../checklists/final-ai-review.md) for the full protocol.

---

### Rule 30 — Prioritize Correctness and Security Over Speed of Delivery
**MUST NOT** sacrifice correctness, security, or established project patterns to deliver a faster or simpler result.

Shipping fast with a vulnerability or a broken edge case is strictly worse than asking for more time or clarification.
