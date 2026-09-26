# Frontend Engineering Rules

> **Scope**: Universal rules for client-side web development across modern frameworks (React, Vue, Svelte, Solid, Next.js) and vanilla JavaScript/TypeScript.

---

## Rule FE-01: Use Native Semantic HTML Elements Before Custom Markup [MUST]
- **Rule**: Interactive controls MUST use native HTML5 tags (`<button>`, `<a>`, `<input>`, `<select>`, `<dialog>`) rather than non-semantic elements (`<div>`, `<span>`) augmented with click handlers.
- **Reason**: Native browser elements provide built-in keyboard accessibility (Tab, Enter, Space), screen reader announcement, and touch target affordances without fragile JavaScript emulation.
- **Risk**: Creating controls that keyboard-only and assistive technology users cannot activate.
- **Correct Approach**:
  ```tsx
  // ❌ BAD: Inaccessible div button
  <div className="btn" onClick={handleSubmit}>Submit</div>

  // ✅ CORRECT: Semantic, accessible button
  <button type="button" onClick={handleSubmit} className="btn">Submit</button>
  ```
- **Exceptions**: Custom canvas or WebGL components where DOM elements do not exist.

---

## Rule FE-02: Lifecycle Effects MUST Have Cleanup Teardowns [MUST]
- **Rule**: Any component lifecycle hook or effect (`useEffect`, `onMounted`) that attaches an event listener, initiates an interval/timer, establishes a WebSocket, or connects an observer MUST return a cleanup function that disposes of that resource upon unmount.
- **Reason**: Without cleanup, subscriptions and listeners persist in browser memory even after component unmounts.
- **Risk**: Memory leaks, state updates on unmounted components, and tab crashes during long sessions.
- **Correct Approach**:
  ```tsx
  // ✅ CORRECT: Listener cleaned up upon unmount
  useEffect(() => {
    const handleResize = () => setDimensions(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  ```
- **Exceptions**: Standard `fetch` calls utilizing an `AbortController` signal wired directly into the browser fetch options, which cancel automatically.

---

## Rule FE-03: Client-Side Form Validation is for UX Only, Never Security [MUST]
- **Rule**: Client-side form validation (HTML `required`, regex patterns, Zod client parsing) MUST be implemented to provide immediate user feedback, but MUST NEVER be treated as a security control. The server MUST independently validate all inputs.
- **Reason**: Any user can bypass client-side validation using `curl`, Postman, or automated scripts.
- **Risk**: Storing unvalidated or malicious data in backend databases due to trusting client checks.
- **Correct Approach**:
  - Frontend: Show inline error hints as the user types or on blur.
  - Backend: Mandate server-side schema parsing on incoming request payloads (see [DATA-PROTECTION-RULES.md](../security/DATA-PROTECTION-RULES.md#rule-dp-04-strict-mass-assignment-protection)).
- **Exceptions**: None.

---

## Rule FE-04: Never Store Sensitive Tokens in localStorage or sessionStorage [MUST]
- **Rule**: Authentication tokens, refresh tokens, and session identifiers MUST NOT be stored in `localStorage`, `sessionStorage`, or window global variables. They MUST be stored in `HttpOnly`, `Secure`, `SameSite` cookies issued by the server.
- **Canonical Reference**: See [AUTHENTICATION-RULES.md](../security/AUTHENTICATION-RULES.md#rule-auth-02-issue-session-tokens-in-secure-httponly-cookies).
- **Reason**: Web storage APIs are completely accessible to any JavaScript running on the domain. Any XSS flaw can read and exfiltrate the token.
- **Risk**: Account takeover via automated token exfiltration.
- **Exceptions**: Non-sensitive UI preferences (dark mode toggle, collapsed sidebar state).

---

## Rule FE-05: Handle All Async Network States Explicitly [MUST]
- **Rule**: Any UI component that fetches asynchronous data MUST provide explicit rendering paths for four states: Idle/Loading, Error, Empty, and Success.
- **Reason**: Applications that ignore loading or error states render blank screens, throw unhandled runtime exceptions, or confuse users.
- **Risk**: High bounce rates and unhandled promise rejections.
- **Correct Approach**:
  ```tsx
  function ProjectList() {
    const { data, isLoading, error, refetch } = useProjects();

    if (isLoading) return <ProjectsSkeleton count={3} />;
    if (error) return <ErrorRecovery message="Failed to load projects" onRetry={refetch} />;
    if (!data || data.length === 0) return <EmptyState title="No projects created yet" />;

    return (
      <ul role="list">
        {data.map(p => <ProjectCard key={p.id} project={p} />)}
      </ul>
    );
  }
  ```
- **Exceptions**: Background prefetching where UI updates are completely invisible to the user.

---

## Rule FE-06: Strict Form Ergonomics & Double-Submit Prevention [MUST]
- **Rule**: Forms MUST use native `<form>` elements with an `onSubmit` handler, inputs MUST have explicit `type`, `autoComplete`, and `name` attributes, and submit buttons MUST be disabled during pending submission to prevent duplicate requests.
- **Reason**: Native forms allow users to submit by pressing Enter, support password managers and autofill, and disabling buttons prevents duplicate order placement.
- **Risk**: Accidental duplicate charges and broken password manager integrations.
- **Exceptions**: Live search-as-you-type filter inputs that update the view without form submission.
