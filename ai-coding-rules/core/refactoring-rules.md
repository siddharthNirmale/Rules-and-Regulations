# Refactoring Rules — Safe AI-Assisted Refactoring Protocol

> **Scope**: Defines the mandatory process AI agents MUST follow when refactoring existing code. Refactoring means improving code structure without changing observable behavior — it does NOT mean rewriting.

---

## Core Principle

> **"Refactor" means change the internals. It MUST NOT mean change the externals, the behavior, or the semantics. "Refactor" MUST NEVER automatically mean "rewrite everything."**

If the external behavior changes, it is not a refactor — it is a feature modification, and it requires explicit specification and review.

---

## Phase 1 — Understand Before Changing

### R1 — Read the Entire Implementation
**MUST** read and understand the complete implementation of the code being refactored before making any change.

- Trace all execution paths, including error paths.
- Identify all external interfaces (exported functions, HTTP endpoints, event names, database schema).
- Identify all consumers of the code being refactored.
- Read the existing tests to understand the specified and verified behavior.

---

### R2 — Identify the Specific Problem
**MUST** clearly identify the concrete problem the refactoring is solving before proceeding.

Valid reasons to refactor:
- Duplication of logic that causes desynchronization.
- A function that violates single responsibility (does two or more unrelated things).
- Variable or function names that actively mislead.
- A module with excessive coupling that makes unit testing impossible.
- A performance bottleneck identified by measurement.

Invalid reasons to refactor:
- "I think this looks cleaner."
- "I would have written it differently."
- "This is an opportunity to add a design pattern."

---

### R3 — Map All Consumers
**MUST** identify every file, function, component, or test that calls, imports, or depends on the code being refactored.

```bash
# Example: find all consumers before refactoring `getUserById`
grep -rn "getUserById" ./src --include="*.ts"
```

Every consumer MUST be updated atomically if the interface changes.

---

## Phase 2 — Establish a Safety Net

### R4 — Confirm Tests Pass Before Starting
**MUST** verify that all existing tests pass before making any refactoring change.

- A failing test before refactoring means there is a pre-existing bug. It is not in scope to fix during refactoring — it MUST be reported and resolved separately.
- Refactoring on a broken baseline makes it impossible to attribute new failures to the refactoring.

---

### R5 — Do Not Change Tests to Make Refactored Code Pass
**MUST NOT** modify existing tests to accommodate a refactoring.

- If a refactoring causes a test to fail, the refactoring broke the external behavior — which means it was not a refactoring.
- Only modify tests when the *specification* has explicitly changed as part of the task.

**Exception**: Pure renaming of internal symbols (not public APIs) may require updating test assertions that reference those symbols directly. This is acceptable only if the observable behavior being tested is unchanged.

---

## Phase 3 — Make the Change

### R6 — Make the Smallest Reasonable Change
**MUST** limit each refactoring step to the smallest independently verifiable improvement.

- Rename a variable → run tests.
- Extract a function → run tests.
- Move a file → run tests.

Do not combine multiple refactoring steps in a single commit without an explicit reason.

---

### R7 — Preserve External Interfaces
**MUST NOT** change any of the following without explicit task specification:

| Interface Type | Examples |
| :--- | :--- |
| Exported function signatures | Parameter names, types, return types |
| HTTP API endpoints | Path, method, request shape, response envelope |
| Event names and payloads | WebSocket events, DOM custom events, message bus topics |
| Database schema | Column names, types, constraints, table names |
| Environment variables | Variable names read from `.env` |
| Public component props | Prop names, types, defaults |

If an external interface MUST change, it is a breaking change — not a refactoring — and MUST be communicated, versioned, and handled appropriately.

---

### R8 — Avoid Unrelated Modifications
**MUST NOT** fix unrelated bugs, rename unrelated variables, reformat unrelated code, or improve unrelated error handling during a refactoring task.

Unrelated improvements MUST be recorded as separate recommendations and addressed in separate, isolated changes.

---

### R9 — Do Not Unnecessary Rewrite
**MUST NOT** replace working, maintainable code with an entirely new implementation because the new version uses a different design pattern or looks different.

- A working `for` loop MUST NOT be replaced with a `reduce` chain unless the reduction is materially more readable.
- A working `if/else` chain MUST NOT be replaced with a strategy pattern unless there are multiple existing and future strategies that justify it.
- A working class-based component MUST NOT be rewritten as a functional component unless this is explicitly requested and the project has migrated.

---

## Phase 4 — Verify After Changing

### R10 — Run the Full Test Suite After Each Change
**MUST** run the complete test suite after each meaningful refactoring step, not just after all steps are complete.

Running tests incrementally localizes failures to the specific change that introduced them.

---

### R11 — Verify Performance and Security Impact
**SHOULD** verify that the refactoring has not introduced:

- Regression in database query performance (check for removed indexes, added N+1 patterns).
- Regression in rendering performance (check for broken memoization, added unnecessary re-renders).
- New security exposure (check that removed abstraction layers have not also removed security controls they were providing).

---

### R12 — Verify Accessibility Is Preserved
**MUST** verify that any UI refactoring has not changed or degraded:

- Semantic HTML structure.
- ARIA attributes and roles.
- Keyboard navigation order and focus management.
- Screen reader announcements.

---

## Phase 5 — Deliver Clean Results

### R13 — Remove Obsolete Code
**SHOULD** remove old implementations, deprecated wrappers, superseded utility functions, and dead import statements that are rendered obsolete by the refactoring.

Clean removals are improvements.

---

### R14 — Ensure the Result Is Easier to Understand Than Before
**MUST** verify that the refactored code is measurably easier to read and understand than the original.

If the result is not clearly more readable, the refactoring has not achieved its purpose and MUST be reconsidered.

A useful test: can a developer unfamiliar with this code understand it in under 2 minutes without additional context?

---

### R15 — Document Why, Not What
**SHOULD** add or preserve comments that explain *why* a non-obvious decision was made.

Remove comments that only restate the obvious code:
```ts
// BAD — restates the code
// Increment counter by 1
counter++;

// GOOD — explains the why
// Optimistic increment before server confirmation; rolled back in the error handler
counter++;
```
