# Code Quality & Clean Craftsmanship Rules

> **Scope**: Readability, cognitive complexity, immutability, naming conventions, and code hygiene.

---

## Rule CQ-01: Low Cognitive Complexity & Guard Clause Returns
- **Rule**: Functions MUST NOT exceed a Cognitive Complexity score of **15** (or a nesting depth exceeding 3 levels). Use early return guard clauses to handle validation failures and edge cases at the top of the function, keeping the main execution path completely un-indented.
- **Reason**: Deeply nested `if/else` pyramids force engineers to maintain a massive mental stack of conditions just to trace one branch of execution.
- **Risk**: Overlooked edge cases, untestable branches, and high defect rates.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Deep arrow anti-pattern
  function processUser(user: User | null) {
    if (user !== null) {
      if (user.isActive) {
        if (user.hasPermission('EDIT')) {
          // 4 levels of indentation
          updateUser(user);
        } else {
          throw new ForbiddenError();
        }
      } else {
        throw new InactiveError();
      }
    } else {
      throw new NotFoundError();
    }
  }

  // ✅ CORRECT: Early return guard clauses (Flat execution)
  function processUser(user: User | null): void {
    if (!user) throw new NotFoundError();
    if (!user.isActive) throw new InactiveError();
    if (!user.hasPermission('EDIT')) throw new ForbiddenError();

    // Clean, un-nested happy path
    updateUser(user);
  }
  ```
- **Exceptions**: Naturally recursive algorithms (e.g. tree traversals, AST visitors).

---

## Rule CQ-02: Descriptive, Intention-Revealing Naming
- **Rule**: Variables, functions, and classes MUST reveal their domain intent. Single-letter variables (except standard `i`, `j` loop indices or `_` for unused parameters) and cryptic abbreviations are strictly prohibited. Boolean variables MUST use auxiliary verb prefixes (`is`, `has`, `can`, `should`).
- **Reason**: Code is read ten times more often than it is written. Cryptic naming (`const u = fetchU(d);`) slows down every subsequent reader and AI agent.
- **Risk**: Misunderstood logic and maintenance bugs.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Cryptic, ambiguous
  const d = 86400;
  const arr = getU(id);
  const flag = arr.length > 0;

  // ✅ CORRECT: Intention-revealing
  const SECONDS_PER_DAY = 86400;
  const activeMemberships = fetchUserMemberships(userId);
  const hasActiveMemberships = activeMemberships.length > 0;
  ```
- **Exceptions**: Well-established mathematical formulas (`x`, `y` coordinates in graphics).

---

## Rule CQ-03: Zero Dead Code or Commented-Out Code Blocks
- **Rule**: Developers and AI agents MUST NOT commit commented-out blocks of code, unused imports, or unreferenced functions.
- **Reason**: Git preserves the complete history of previous code implementations. Commented-out code litters files, creates cognitive noise, and rots immediately as surrounding APIs change.
- **Risk**: Confusion over whether the commented code is an intentional fallback or forgotten trash.
- **Correct Approach**:
  - Delete obsolete code completely. If you need it in the future, retrieve it from `git log`.
  - Configure linters with `no-unused-vars` and `unused-imports/no-unused-imports` as blocking errors.
- **Exceptions**: None.

---

## Rule CQ-04: Eliminate Magic Numbers & Magic Strings
- **Rule**: Hardcoded numbers or strings with business significance MUST be extracted into named constants or TypeScript enums/unions.
- **Reason**: Numbers like `86400000`, `1.15`, or status strings like `"REV_PENDING"` convey no meaning on their own and cannot be updated consistently across a codebase.
- **Risk**: Inconsistent values when updating business rules (e.g. updating a tax rate in 3 places but missing a 4th).
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Magic number
  if (elapsedTime > 604800000) { expireSession(); }

  // ✅ CORRECT: Named domain constant
  const SEVEN_DAYS_IN_MS = 7 * 24 * 60 * 60 * 1000;
  if (elapsedTime > SEVEN_DAYS_IN_MS) { expireSession(); }
  ```
- **Exceptions**: Natural numbers `0`, `1`, `-1` used in simple loops, offset counters, or array index checks.

---

## Rule CQ-05: Default to Immutability
- **Rule**: Use `const` by default for all variable declarations. Avoid `let` unless a variable is explicitly reassigned. Treat objects and arrays as immutable; avoid mutating methods like `.push()`, `.splice()`, or `.reverse()` on shared objects. Use spread operators or non-mutating equivalents (`.concat()`, `.slice()`, `.map()`, `.filter()`).
- **Reason**: Shared mutable state is the leading cause of spooky action-at-a-distance bugs where modifying an object in one function inadvertently corrupts it in another.
- **Risk**: Hard-to-trace bugs, broken caching, and reactivity failures in modern UI frameworks.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Mutates original array in place
  function addRole(user: User, role: string) {
    user.roles.push(role);
  }

  // ✅ CORRECT: Returns a new immutable copy
  function addRole(user: User, role: string): User {
    return {
      ...user,
      roles: [...user.roles, role]
    };
  }
  ```
- **Exceptions**: Tight mathematical loops where temporary in-place mutation provides proven 10x performance gains documented by benchmarks.
