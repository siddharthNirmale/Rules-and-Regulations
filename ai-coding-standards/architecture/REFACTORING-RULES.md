# Safe AI Refactoring Rules & Protocols

> **Core Principle**: Refactoring means improving the internal design of existing code without changing its observable external behavior. "Refactor" MUST NEVER mean "rewrite everything from scratch."

---

## Rule REF-01: Verify Passing Baseline Before Touching Code [MUST]
- **Rule**: Before modifying any code during a refactoring task, the developer or AI agent MUST run the existing test suite and confirm that all relevant unit and integration tests pass cleanly. If tests fail initially, the baseline failure MUST be resolved or documented before refactoring begins.
- **Reason**: If tests are already failing before you begin refactoring, you cannot know whether a test failure after your change was caused by your edits or pre-dated them.
- **Risk**: Blaming refactoring for pre-existing bugs or masking new regressions under existing noise.
- **Correct Approach**:
  ```bash
  # Step 1: Run baseline tests
  npm test -- tests/unit/payment-service.test.ts
  # If all PASS, proceed with refactor.
  ```
- **Exceptions**: Legacy codebases with zero existing tests (in which case Rule REF-02 applies).

---

## Rule REF-02: Write Characterization Tests Before Refactoring Untested Code [MUST]
- **Rule**: If the code to be refactored lacks automated test coverage, the developer or AI agent MUST write "Characterization Tests" that capture and lock in the current observable behavior (including quirky edge cases) before restructuring the code.
- **Reason**: Without tests asserting current behavior, any structural change risks silently altering business logic.
- **Risk**: Undetected breaking changes in production.
- **Correct Approach**:
  - Write test cases asserting the exact output for typical, empty, and boundary inputs.
  - Verify characterization tests pass against the *original* code.
  - Perform the refactor.
  - Verify characterization tests *still pass* against the *refactored* code.
- **Exceptions**: Truly dead code marked for immediate deletion.

---

## Rule REF-03: Mandatory Downstream Consumer Mapping [MUST]
- **Rule**: Before changing any function name, parameter order, return type, or module export, the developer or AI agent MUST search the entire repository to find and list all callers/consumers.
- **Reason**: Renaming an exported function or altering its parameters without updating all callers causes runtime `TypeError` crashes in un-exercised code paths.
- **Risk**: Broken builds and production regressions.
- **Correct Approach**:
  ```bash
  # Grep across codebase to locate all consumers
  git grep -n "calculateDiscount" src/
  ```
  - If 5 files call the function, either:
    1. Update all 5 callers atomically in the same commit, OR
    2. Maintain backward compatibility by keeping the original function as a wrapper around the new implementation.
- **Exceptions**: Internal private helper functions whose scope is 100% contained within a single file.

---

## Rule REF-04: Never Mix Refactoring with Feature Modifications [MUST]
- **Rule**: Pure refactoring commits MUST NOT contain new features, bug fixes, or business logic alterations. Conversely, feature branches MUST NOT perform opportunistic massive code reformatting.
- **Reason**: Mixing refactoring with behavior changes makes code reviews nearly impossible. Reviewers cannot distinguish which line in a 500-line diff represents a deliberate business logic change and which is just a cosmetic rename.
- **Risk**: Bugs slip through code review unnoticed, and rollbacks become high-risk all-or-nothing operations.
- **Correct Approach**:
  - Commit 1: `refactor(auth): extract token verification into separate service` (Zero behavior change).
  - Commit 2: `feat(auth): add support for WebAuthn passkeys` (Feature addition).
- **Exceptions**: None.

---

## Rule REF-05: Surgical Diffs & Zero Diff Pollution [MUST]
- **Rule**: When editing an existing file, the AI agent MUST modify ONLY the targeted lines. Do NOT reformat unrelated functions, do NOT alter unrelated indentation, and do NOT delete pre-existing explanatory comments.
- **Reason**: Full-file rewrites destroy `git blame` history, make diffs unreadable, and introduce merge conflicts across parallel feature branches.
- **Risk**: Accidental deletion of subtle edge-case fixes and severe merge conflicts.
- **Correct Approach**: Use localized replacement chunks targeting exact line ranges. Never replace a 400-line file to change 3 lines.
- **Exceptions**: Running a project-wide code formatter (Prettier/Ruff) in a dedicated, standalone formatting commit.
