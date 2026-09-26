# Backend Engineering Rules

> **Scope**: Layered architecture, business logic isolation, event loop safety, concurrency control, and outbound network discipline.

---

## Rule BE-01: Strict Layered Architecture (Separation of Concerns) [MUST]
- **Rule**: Backend applications MUST segregate responsibilities into three distinct, decoupled layers:
  1. **Transport Layer (Controllers/Routes)**: Handles HTTP request parsing, status codes, and response serialization. MUST NOT contain business rules or raw database queries.
  2. **Domain Layer (Services)**: Contains business rules, validations, calculations, and orchestrations. MUST NOT reference transport objects (`req`, `res`).
  3. **Persistence Layer (Repositories/DAOs)**: Executes queries against databases, caches, or external third-party storage.
- **Reason**: Decoupled layers enable unit testing of domain logic in isolation without mocking HTTP servers or live database connections.
- **Risk**: Monolithic controllers that cannot be unit tested and cause regressions when APIs evolve.
- **Correct Approach**:
  - Keep controllers thin (< 30 lines).
  - Pass clean DTOs (Data Transfer Objects) to services.
  - Return plain typed entities from repositories.
- **Exceptions**: Trivial proxy passes or healthcheck routes (`/healthz`).

---

## Rule BE-02: Never Block the Asynchronous Event Loop [MUST]
- **Rule**: In asynchronous runtimes (Node.js, Python asyncio), developers MUST NOT execute long-running synchronous CPU-bound operations (e.g. synchronous crypto, massive JSON parsing, un-chunked file compression) on the main thread. Offload to worker threads or background queues.
- **Reason**: The event loop executes on a single thread. Blocking it halts request processing for all concurrent users, causing severe latency spikes and connection timeouts.
- **Risk**: Total application unresponsiveness and Denial of Service (DoS) under moderate load.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Synchronous PBKDF2 blocks the event loop for 150ms
  const hash = crypto.pbkdf2Sync(password, salt, 100000, 64, 'sha512');

  // ✅ CORRECT: Asynchronous offload to thread pool
  const hash = await crypto.promises.pbkdf2(password, salt, 100000, 64, 'sha512');
  ```
- **Exceptions**: Initialization scripts that execute once prior to binding the server port.

---

## Rule BE-03: Guard State Mutations Against Concurrency & Race Conditions [MUST]
- **Rule**: Multi-step state modifications (e.g. balance deductions, inventory decrements, seat reservations) MUST be executed inside atomic database transactions using row-level locks (`SELECT ... FOR UPDATE`), serializable isolation, or atomic decrement operators (`UPDATE items SET stock = stock - 1 WHERE stock >= 1`).
- **Canonical Reference**: See [DATABASE-RULES.md](./DATABASE-RULES.md).
- **Reason**: Concurrent requests arriving milliseconds apart read the same initial state before either write commits, leading to double-spending or overselling.
- **Risk**: Financial loss, oversold inventory, and corrupted system state.
- **Exceptions**: Append-only event logs or immutable audit streams.

---

## Rule BE-04: Mandatory Timeouts on All Outbound Network Requests [MUST]
- **Rule**: Every outgoing HTTP request, database query, or remote service call MUST declare an explicit timeout (e.g. 5 seconds for HTTP, 2 seconds for database queries).
- **Reason**: If an external dependency hangs or experiences packet loss, client sockets remain open indefinitely without timeouts, rapidly exhausting server connection pools.
- **Risk**: Cascading failure that brings down the entire application cluster.
- **Correct Approach**:
  ```typescript
  // ✅ CORRECT: AbortSignal enforces 5-second maximum timeout
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), 5000);

  try {
    const res = await fetch('https://api.external.com/v1/resource', {
      signal: controller.signal
    });
  } finally {
    clearTimeout(timeoutId);
  }
  ```
- **Exceptions**: Server-Sent Events (SSE) or long-polling streams where long-lived connections are intentional.
