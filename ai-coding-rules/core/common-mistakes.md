# Common AI Coding Mistakes — Root Causes & Prevention Rules

> **Scope**: Documents 24 canonical failure patterns observed in AI-generated code. For each: the root cause, why it is harmful, and the specific enforceable rule that prevents it.

---

## Mistake 1 — Over-Engineered Code

**What happens**: The AI generates abstract base classes, factory registries, plugin systems, or complex generic types to solve a problem that a simple function would handle.

**Root cause**: Training data biases toward "impressive" enterprise patterns. The model conflates sophistication with correctness.

**Why it is harmful**: Over-engineered code is harder to read, harder to debug, harder to test, and harder to delete. It introduces accidental complexity that compounds over time.

**Prevention rule**: Implement the simplest solution that is correct today. Add abstractions only when a second genuine use case exists. See [`core/general-rules.md` Rule 5](./general-rules.md).

---

## Mistake 2 — Excessive JavaScript / TypeScript

**What happens**: The AI implements complex client-side state machines, custom scroll effects, DOM manipulation, or layout logic that could be achieved with CSS or HTML attributes.

**Root cause**: The AI defaults to the language it is most fluent in for the problem context.

**Why it is harmful**: JavaScript is the most fragile and expensive layer of the frontend stack. Unnecessary JS inflates bundle size, increases parse time, risks accessibility failures, and introduces runtime failure modes that CSS does not have.

**Prevention rule**: Prefer CSS and HTML-native solutions (`:focus-visible`, `details`/`summary`, `scroll-behavior`, CSS animations) before reaching for JavaScript. JS is for behavior, not layout.

---

## Mistake 3 — Duplicate Logic

**What happens**: The AI writes a new date formatter, error boundary, API request helper, or validation function that already exists elsewhere in the codebase.

**Root cause**: The model processes the immediate context but does not always search the wider codebase for existing implementations.

**Why it is harmful**: Duplicate logic drifts out of sync, leading to inconsistent behavior between two code paths that should be identical. Finding and maintaining two copies is double the cost.

**Prevention rule**: Search for existing implementations before writing new code. See [`core/general-rules.md` Rule 6](./general-rules.md).

---

## Mistake 4 — Unnecessary Dependencies

**What happens**: The AI adds a npm package for a 5-line utility function (e.g., `left-pad`, `is-odd`, `uuid` when `crypto.randomUUID()` is available natively).

**Root cause**: The model has seen these packages used in training examples without awareness of their total cost.

**Why it is harmful**: Every dependency adds: attack surface, supply-chain risk, bundle size, maintenance obligation, and version conflict potential.

**Prevention rule**: Justify every dependency against the criteria in [`core/general-rules.md` Rule 10](./general-rules.md). Prefer standard library and already-present dependencies.

---

## Mistake 5 — Poor Component Architecture

**What happens**: A React component handles data fetching, business logic, form state, and rendering all in one 400-line file. Or the opposite: trivial logic is fragmented into a dozen micro-components with excessive prop-drilling.

**Root cause**: The AI generates based on the immediate prompt without planning the broader component graph.

**Why it is harmful**: Monolithic components are untestable and unmaintainable. Over-fragmented components make data flow opaque and create unnecessary render overhead.

**Prevention rule**: Separate concerns — presentation from logic from data fetching. Follow the project's established component conventions. See [`frontend/react-rules.md`](../frontend/react-rules.md).

---

## Mistake 6 — Inconsistent Naming

**What happens**: The AI mixes camelCase, snake_case, and PascalCase within the same module. Or uses `getUser`, `fetchUserById`, and `loadCurrentUser` for semantically identical operations.

**Root cause**: Without a full codebase context, the model borrows naming from unrelated training examples.

**Why it is harmful**: Inconsistent naming creates cognitive friction, complicates search/refactoring, and signals poor code ownership.

**Prevention rule**: Adopt the naming convention already established in the project. When in doubt, match the nearest file in the same directory. Names should be predictable and consistent.

---

## Mistake 7 — Poor Error Handling

**What happens**: Empty `catch {}` blocks, catch blocks that log only `error.message` (losing stack trace), or error handling that swallows the error and returns a successful status code.

**Root cause**: The model generates the "happy path" first and appends error handling as an afterthought, often using a minimal boilerplate.

**Why it is harmful**: Poor error handling causes silent data corruption, leaves users with broken UI states, and makes debugging in production impossible.

**Prevention rule**: Every thrown exception MUST be caught at an appropriate boundary with explicit handling. Log full error objects (not just `.message`). Return correct HTTP status codes. See [`backend/error-handling.md`](../backend/error-handling.md).

---

## Mistake 8 — Missing Edge Cases

**What happens**: The generated function works for typical inputs but crashes or misbehaves on: empty strings, null values, empty arrays, very large numbers, negative numbers, concurrent invocations, or boundary values.

**Root cause**: The model optimizes for the dominant path in training data (typical happy-path inputs).

**Why it is harmful**: Edge cases are where security vulnerabilities and production incidents live. Null-pointer equivalents crash processes. Integer overflows corrupt data. Empty arrays break UI layout.

**Prevention rule**: For every non-trivial function, explicitly reason about null, empty, zero, boundary-max, and concurrent inputs before committing. See [`core/general-rules.md` Rule 18](./general-rules.md).

---

## Mistake 9 — Breaking Existing Functionality During Refactoring

**What happens**: A refactoring that renames a shared utility, changes a function signature, or restructures a module causes failures in consumers that were not updated.

**Root cause**: The model focuses on the immediate change without mapping all consumers.

**Why it is harmful**: Production regressions. Broken builds. Emergency rollbacks. Eroded trust in AI-assisted development.

**Prevention rule**: Before any refactoring, identify all consumers of the changed interface. Update all consumers atomically. Run the full test suite before delivery. See [`core/refactoring-rules.md`](./refactoring-rules.md).

---

## Mistake 10 — Overwriting Unrelated Code

**What happens**: When editing a file, the AI reformats surrounding code, removes unrelated comments, or changes variable names outside the scope of the task.

**Root cause**: The model generates a full-file rewrite when given a file as context, applying its own style preferences.

**Why it is harmful**: Diff pollution, accidental removal of meaningful comments, merge conflicts, and unpredictable behavior in unreviewed sections.

**Prevention rule**: Make surgical, minimal edits. Only modify the lines required by the task. Preserve all existing comments, formatting, and code outside the change boundary.

---

## Mistake 11 — Poor Accessibility

**What happens**: Interactive elements lack labels, buttons use `<div>` instead of `<button>`, forms have no error association, color is used as the sole differentiator.

**Root cause**: Accessibility is non-visual and rarely caught in visual previews. Training examples skew toward functional-looking code that may fail assistive technology.

**Why it is harmful**: Excludes users with disabilities. Creates legal liability (WCAG compliance is legally required in many jurisdictions). Fails screen reader users and keyboard-only users entirely.

**Prevention rule**: Use semantic HTML by default. Always associate labels with inputs. Never use color as the sole indicator. Test with a keyboard. See [`frontend/accessibility.md`](../frontend/accessibility.md).

---

## Mistake 12 — Poor Responsive Behavior

**What happens**: Fixed-width containers, pixel-absolute sizes, or desktop-only layouts that break on mobile. Or conversely, mobile-only patterns that don't scale up to desktop.

**Root cause**: The model generates for a single viewport context based on the dominant training example.

**Why it is harmful**: Over 60% of web traffic is mobile. A layout broken on mobile represents a broken product for the majority of users.

**Prevention rule**: Use fluid layouts (CSS Grid, Flexbox, `min()`, `max()`, `clamp()`). Design mobile-first. Test at 360px, 768px, and 1280px. See [`ui-ux/responsive-design.md`](../ui-ux/responsive-design.md).

---

## Mistake 13 — Memory Leaks

**What happens**: Event listeners added in `useEffect` without a cleanup function. `setInterval` timers that continue after a component unmounts. WebSocket connections that are never closed.

**Root cause**: The model generates setup code without pairing it with teardown logic.

**Why it is harmful**: Memory leaks degrade performance over time in long-running sessions and can crash browser tabs or Node.js processes.

**Prevention rule**: Every resource acquisition (event listener, timer, subscription, connection) MUST have a corresponding cleanup/teardown path. In React, every `useEffect` that subscribes MUST return a cleanup function.

```ts
useEffect(() => {
  const handler = (e: Event) => handleResize(e);
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler); // ✅ cleanup
}, []);
```

---

## Mistake 14 — Performance Problems

**What happens**: N+1 database queries, missing indexes, large synchronous JSON operations blocking the event loop, unbounded array operations on large datasets, re-rendering entire trees on minor state changes.

**Root cause**: The model generates for correctness first and rarely considers the performance cost of its access patterns.

**Why it is harmful**: Performance problems compound. An N+1 that is tolerable at 100 users is catastrophic at 100,000 users.

**Prevention rule**: For any database access in a loop, consider batching or JOINs. Profile before optimizing. Do not introduce N+1 patterns. See [`performance/performance-rules.md`](../performance/performance-rules.md).

---

## Mistake 15 — Security Vulnerabilities (General)

**What happens**: SQL injection via string interpolation, XSS via `innerHTML`, command injection via unsanitized `exec()` arguments, path traversal via unvalidated file paths.

**Root cause**: The model optimizes for conciseness and often uses the shortest code path, which frequently bypasses parameterization or encoding.

**Why it is harmful**: Direct route to data breaches, account takeover, and server compromise.

**Prevention rule**: Never interpolate user input into SQL, shell commands, file paths, or HTML. Always use parameterized queries, encoding functions, and allowlist validation. See [`security/security-rules.md`](../security/security-rules.md).

---

## Mistake 16 — Hardcoded Secrets or Credentials

**What happens**: API keys, database passwords, JWT secrets, or OAuth client secrets appear directly in source code.

**Root cause**: The fastest way to make code run is to hardcode the value seen in the developer's environment.

**Why it is harmful**: Source code is frequently committed to version control and may be public or broadly accessible. A hardcoded secret is a permanent exposure event.

**Prevention rule**: Secrets MUST be loaded from environment variables or a secrets manager. Source code MUST be scanned for secret patterns before commit. If a secret is accidentally committed, it MUST be rotated immediately regardless of repository visibility.

---

## Mistake 17 — Incorrect Authentication / Authorization Logic

**What happens**: Authentication checks performed on the client only. Authorization checks missing from specific API routes. JWT signatures not verified. Session tokens stored in `localStorage`.

**Root cause**: The model may implement the "shape" of auth logic without fully understanding the trust model, or may copy patterns that are appropriate in one context but insecure in another.

**Why it is harmful**: Incorrect authentication allows account impersonation. Incorrect authorization allows privilege escalation and data exposure.

**Prevention rule**: Authentication and authorization MUST be enforced on the server for every protected route. JWT signatures MUST be cryptographically verified. Session tokens MUST use `HttpOnly`, `Secure`, `SameSite` cookie attributes. See [`security/authentication.md`](../security/authentication.md) and [`security/authorization.md`](../security/authorization.md).

---

## Mistake 18 — Unsafe Data Handling

**What happens**: Raw user data returned directly to API consumers (including internal fields, passwords, salts, relationship data). PII logged in plain text. Unencrypted sensitive data at rest.

**Root cause**: The model returns the database object directly as the API response without an explicit serialization/projection layer.

**Why it is harmful**: Data exposure is a primary security and compliance risk. Password hashes, internal IDs, and confidential fields exposed in API responses have caused major breaches.

**Prevention rule**: Always project/serialize explicitly. Never return raw ORM objects. Never log passwords, tokens, or credit card data. See [`security/data-security.md`](../security/data-security.md).

---

## Mistake 19 — Poor API Design

**What happens**: Inconsistent response envelopes. Non-standard HTTP status codes (e.g., returning 200 for validation errors). Missing pagination on unbounded collection endpoints. No versioning strategy.

**Root cause**: The model generates APIs that work for the immediate use case without considering the client consumption experience or long-term compatibility.

**Why it is harmful**: Poor API contracts make client development difficult and make version migrations costly. Returning 200 for errors silently breaks error handling in consumers.

**Prevention rule**: Follow REST conventions strictly. Use standard HTTP status codes. Provide consistent response envelopes. Paginate all collection endpoints. See [`backend/api-rules.md`](../backend/api-rules.md).

---

## Mistake 20 — Improper Validation

**What happens**: Client-side-only validation. Missing length limits. No type enforcement. No format validation (email, UUID, date). Validation only on the "create" path but not the "update" path.

**Root cause**: Validation is perceived as boilerplate and is often the last thing generated. Client-side validation is visible and easy to demonstrate.

**Why it is harmful**: Missing server-side validation is an injection and data-integrity vulnerability. Inconsistent validation between create/update paths leads to corrupt database states.

**Prevention rule**: All validation MUST be enforced server-side. Schema validation MUST be applied on every mutation endpoint. Field length, type, and format MUST be enforced.

---

## Mistake 21 — Inadequate Logging and Monitoring

**What happens**: No structured logging. Events logged only to `console.log` without severity levels. Security-sensitive events (failed logins, permission denials, admin actions) not logged. Log messages containing PII or secrets.

**Root cause**: Logging feels secondary to the feature and is often skipped or reduced to minimal output.

**Why it is harmful**: Without adequate logging, security incidents are undetectable. Debugging production failures becomes guesswork. Compliance requirements for audit trails go unmet.

**Prevention rule**: Use structured logging with severity levels. Log all security-relevant events. Never log secrets, tokens, or raw PII. See [`backend/error-handling.md`](../backend/error-handling.md).

---

## Mistake 22 — Overuse of Abstractions

**What happens**: Generic utility layers, pluggable adapter interfaces, and inversion-of-control containers that add four layers of indirection between a business action and its implementation.

**Root cause**: Enterprise patterns from the training corpus are applied indiscriminately regardless of project scale.

**Why it is harmful**: Premature abstraction is harder to remove than concrete code. It makes reading, debugging, and testing significantly harder.

**Prevention rule**: Introduce abstraction only when the cost of repetition exceeds the cost of the abstraction. Three identical implementations is the typical threshold for justified extraction.

---

## Mistake 23 — Dependency Proliferation

**What happens**: Adding 8 new packages to implement a feature that could be done with 2 built-in platform APIs and one well-chosen library.

**Root cause**: The model's training data includes examples with heavy dependency usage that was reasonable in those specific projects.

**Why it is harmful**: Each dependency is a potential security vulnerability, a maintenance obligation, and a bundle size contributor. Projects with 1,000+ direct dependencies are increasingly fragile.

**Prevention rule**: Before adding any dependency, verify it cannot be replaced by a native platform API, an existing project dependency, or a short implementation. See [`core/general-rules.md` Rule 10](./general-rules.md).

---

## Mistake 24 — "Magic" Code That Is Difficult to Understand or Maintain

**What happens**: Dynamic property access via computed keys, metaprogramming via `eval`, complex regex without explanation, deeply nested ternaries, or "clever" one-liners that accomplish something surprising.

**Root cause**: The model optimizes for brevity and novelty over clarity. It has absorbed many "code golf" examples.

**Why it is harmful**: Magic code cannot be safely modified. It fails in ways that are nearly impossible to diagnose. It cannot be reviewed by a human in under a minute, which means it cannot be safely maintained.

**Prevention rule**: Write boring, explicit, readable code. If a line requires a 3-sentence comment to explain it, rewrite it as 3 readable lines instead. Prefer clarity over cleverness, always.

```js
// BAD — magic
const result = data?.user?.roles?.includes?.('admin') ?? false;
users.filter(u => +!!u?.active && u?.roles?.some(r => ['admin','mod'].includes(r)));

// GOOD — explicit
const isAdmin = user !== null && user.roles.includes('admin');
const activeAdminOrMod = users.filter(user => {
  return user.active === true && (user.roles.includes('admin') || user.roles.includes('mod'));
});
```
