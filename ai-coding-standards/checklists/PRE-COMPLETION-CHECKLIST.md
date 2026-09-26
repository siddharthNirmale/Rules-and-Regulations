# Pre-Commit & Pre-Completion Verification Checklist

> **Authority**: Mandatory Gate  
> **Target**: Every AI coding agent, autonomous agent workflow, or pull-request submission.  
> **Requirement**: An AI agent **MUST NOT** state "I am done" or declare a task complete until every applicable item in this checklist has been verified and passed.

---

## 1. Quick Gate Execution (Commands)

Before reporting task completion, run the following verification pipeline in your project root:

```bash
# 1. Typecheck & Linting
npm run typecheck || npx tsc --noEmit
npm run lint || npx eslint .

# 2. Automated Test Suite (Must have 0 failures)
npm test || pytest || cargo test || go test ./...

# 3. Security & Dependency Audit
npm audit --production || pip-audit || cargo audit

# 4. Git Diff Inspection (Zero unintended changes)
git status --short
git diff --stat
```

---

## 2. The 17-Point Verification Checklist

### 1. Functionality & Requirements [MUST]
- [ ] **Requirements Met**: All explicit user requirements and user stories are fully implemented.
- [ ] **No Dead Code**: No unused variables, uncalled helper functions, or dangling imports introduced.
- [ ] **Observable Correctness**: Logic was manually or automatically verified against real expected input and output data.

### 2. Security Posture [MUST]
- [ ] **No Secrets Exposed**: Source code, test fixtures, and commit messages contain zero API keys, passwords, bearer tokens, or private certificates.
- [ ] **Zero Trust Input**: All incoming data (parameters, queries, headers, bodies) is treated as untrusted.
- [ ] **Injection Defenses**: All database queries are 100% parameterized. Zero string interpolation in queries or shells.
- [ ] **Safe HTML Rendering**: No unescaped user-generated content rendered via `dangerouslySetInnerHTML`, `v-html`, or `innerHTML`.

### 3. Authentication & Authorization [MUST]
- [ ] **Server-Side Enforcement**: Permissions and roles are verified on the server for every new/modified route.
- [ ] **IDOR / BOLA Prevention**: Resource queries explicitly filter by tenant ID or user ID (e.g., `WHERE id = :id AND user_id = :userId`).
- [ ] **Cookie Flags**: Auth cookies configured with `HttpOnly`, `Secure`, and `SameSite=Lax/Strict`.

### 4. Input Validation & Sanitization [MUST]
- [ ] **Schema Validation**: Request payloads are validated against strict schemas (Zod, Pydantic, Joi).
- [ ] **Boundary Limits**: Numeric ranges, string lengths, and array capacities have explicit upper bounds.
- [ ] **Strip Unknown Keys**: Disallowed or unexpected JSON payload keys are automatically rejected or stripped.

### 5. Error Handling & Resilience [MUST]
- [ ] **Zero Silent Swallowing**: No empty `catch {}` blocks. Every exception is logged or handled cleanly.
- [ ] **Sanitized Client Errors**: Internal stack traces, database schema names, and file paths are never returned to clients in production.
- [ ] **Correct HTTP Status Codes**: Semantic status codes returned (400 for bad input, 401 for unauth, 403 for forbidden, 404 for not found, 409 for conflict).

### 6. Accessibility (a11y) [MUST]
- [ ] **Semantic Markup**: Native elements used (`<button>`, `<a>`, `<input>`) instead of generic `<div>` clickables.
- [ ] **Keyboard Navigable**: All interactive components are reachable via `Tab` and activated via `Enter` / `Space`.
- [ ] **Focus Visibility**: `:focus-visible` styling is preserved or replaced with high-contrast outlines.
- [ ] **Form Labels**: Every input field has an associated `<label>` or `aria-label`.

### 7. Responsive Behavior [MUST]
- [ ] **Fluid Layouts**: No fixed pixel widths (`width: 1200px`) causing horizontal scrollbars on mobile viewports.
- [ ] **Touch Target Size**: Interactive buttons and links meet the minimum 44x44 CSS pixel footprint.
- [ ] **No Hover Exclusivity**: Features are accessible on touchscreens without requiring mouse hover.

### 8. Performance & Efficiency [MUST]
- [ ] **No N+1 Queries**: Database queries inside loops are replaced with batched queries or joins.
- [ ] **Media Optimization**: Images use responsive attributes, lazy loading (`loading="lazy"`), and explicit dimensions.
- [ ] **Bundle Discipline**: No heavy libraries added for features achievable with standard native APIs.

### 9. Code Quality & Simplicity [MUST]
- [ ] **No Over-Engineering**: Simplest correct implementation used. No unnecessary generic factories or multi-tier abstractions.
- [ ] **No Duplication**: Reused existing project helpers and components rather than writing duplicates.
- [ ] **Naming Consistency**: Follows existing repository naming conventions (casing, file extensions, architectural patterns).

### 10. Dependencies [MUST]
- [ ] **Justified Packages**: Any newly added package is strictly necessary, actively maintained, and free of known high/critical CVEs.
- [ ] **Lockfile Updated**: `package-lock.json`, `pnpm-lock.yaml`, or equivalent is updated and committed alongside code changes.

### 11. Secrets Management [MUST]
- [ ] **Environment Segregation**: Secrets loaded from environment variables or secrets managers.
- [ ] **No Insecure Defaults**: Fallback values like `process.env.SECRET || 'default123'` are strictly prohibited.
- [ ] **Gitignore Verified**: New sensitive files (`.env`, `.pem`, local sqlite files) are ignored in `.gitignore`.

### 12. Automated Tests [MUST]
- [ ] **Baseline Passing**: All pre-existing test suites continue to pass with zero failures.
- [ ] **New Tests Added**: Positive, negative, and edge-case unit/integration tests added for new functionality.
- [ ] **Deterministic Execution**: Tests execute reliably without depending on live external networks or system clocks.

### 13. Build & Static Analysis [MUST]
- [ ] **Clean Build**: `npm run build` or framework compile command finishes with exit code 0.
- [ ] **Zero Type Errors**: TypeScript or typechecker passes without `@ts-ignore` or unchecked `any`.
- [ ] **Zero Lint Warnings**: Linter passes cleanly without suppressing warnings without justification.

### 14. Production Configuration [MUST]
- [ ] **Debug Flags Disabled**: Development logging, test harnesses, and mock flags disabled in production environments.
- [ ] **Graceful Shutdown**: Long-running servers trap `SIGTERM` / `SIGINT` to drain in-flight connections.

### 15. Documentation [MUST]
- [ ] **ADR Recorded**: If an architectural or security change was made, an Architecture Decision Record or comment documents the rationale.
- [ ] **API Specs Updated**: OpenAPI / Swagger definitions or TypeScript contract types reflect any added or modified endpoints.

### 16. Regression Risks [MUST]
- [ ] **Public Signatures Intact**: Exported functions, endpoints, and database column names retain backward compatibility.
- [ ] **Downstream Callers Checked**: All consumers of modified functions across the codebase were searched and confirmed unbroken.

### 17. Unintended Changes (Diff Hygiene) [MUST]
- [ ] **Diff Scope Inspected**: `git diff` reviewed line-by-line. No accidental whitespace-only changes, commented-out dead code, or unrelated file modifications exist.
- [ ] **Comments Preserved**: Pre-existing explanatory comments and docstrings in touched files were preserved.

---

## 3. The 13 AI Self-Review Reflection Questions

Before declaring done, the AI agent MUST self-verify:
1. Did I understand the requested change?
2. Did I inspect the existing implementation?
3. Did I modify anything unrelated?
4. Did I introduce unnecessary dependencies?
5. Did I introduce unnecessary complexity?
6. Could this change create a security vulnerability?
7. Did I validate external/user-controlled input?
8. Did I handle errors and edge cases?
9. Did I preserve existing functionality?
10. Did I test the relevant behavior?
11. Did I check the production build?
12. Did I review the final changes?
13. Am I making assumptions that should instead be verified?

---

## 4. Failure Protocol

If **any** single item above fails:
1. **DO NOT** claim the task is complete.
2. **DO NOT** commit or push the code.
3. Fix the specific deficiency immediately.
4. Re-run the verification pipeline from Step 1.
