# Production Deployment & Container Hygiene Rules

> **Scope**: Container security, zero-downtime deployment, graceful process termination, health probes, and migration orchestration.

---

## Rule DEP-01: Run Containers as Dedicated Non-Root Users
- **Rule**: Dockerfiles and container specifications MUST declare an unprivileged system user (`USER appuser` or `USER node`) before the runtime entrypoint. Containers MUST NOT execute as `root`.
- **Reason**: If an attacker achieves Remote Code Execution (RCE) inside a container running as root, any container-escape vulnerability grants root access to the entire host machine.
- **Risk**: Total host and Kubernetes cluster compromise.
- **Correct Approach**:
  ```dockerfile
  # ✅ CORRECT: Secure container execution
  FROM node:20-alpine AS runner
  WORKDIR /app

  # Create unprivileged system user and group
  RUN addgroup -S appgroup && adduser -S appuser -G appgroup
  USER appuser

  COPY --chown=appuser:appgroup dist/ ./dist
  CMD ["node", "dist/server.js"]
  ```
- **Exceptions**: Specific system utility containers (e.g., VPN clients or storage mount daemons) that strictly require host capabilities.

---

## Rule DEP-02: Implement Distinct Liveness & Readiness Probes
- **Rule**: Every production service MUST expose distinct HTTP endpoints for container orchestration:
  1. **Liveness Probe (`/healthz`)**: Checks if the process is responsive and not locked in a deadlock.
  2. **Readiness Probe (`/readyz`)**: Checks if external dependencies (database, Redis, queues) are connected and the app is ready to accept user traffic.
- **Reason**: Kubernetes or load balancers use readiness probes to avoid routing user traffic to newly spawned pods before their database connection pools are warmed up.
- **Risk**: 502 Bad Gateway errors experienced by users during rolling updates.
- **Correct Approach**:
  ```typescript
  // Liveness: Process is alive
  app.get('/healthz', (req, res) => res.status(200).send('OK'));

  // Readiness: All dependencies verified healthy
  app.get('/readyz', async (req, res) => {
    const isDbConnected = await checkDatabaseConnection();
    const isRedisConnected = await checkRedisConnection();

    if (isDbConnected && isRedisConnected) {
      return res.status(200).json({ status: 'READY' });
    }
    return res.status(503).json({ status: 'NOT_READY' });
  });
  ```
- **Exceptions**: Stateless serverless functions (AWS Lambda) where execution lifecycle is managed by the cloud runtime.

---

## Rule DEP-03: Graceful Termination on `SIGTERM` / `SIGINT`
- **Rule**: Backend processes MUST listen for OS termination signals (`SIGTERM`, `SIGINT`), stop accepting new incoming HTTP connections, wait for in-flight requests to complete (with a 10-30s grace timeout), close open database sockets, and then exit cleanly with code `0`.
- **Reason**: During zero-downtime rolling deploys, orchestrators send `SIGTERM` to decommission old pods. Abruptly killing the process terminates active customer checkout transactions mid-flight.
- **Risk**: Customer transaction truncation, orphaned database locks, and data inconsistency.
- **Correct Approach**:
  ```typescript
  // ✅ CORRECT: Graceful shutdown handler
  const server = app.listen(PORT);

  function gracefulShutdown(signal: string) {
    logger.info(`Received ${signal}. Draining connections...`);
    
    server.close(async () => {
      logger.info('HTTP server closed. Draining database pools...');
      await db.pool.end();
      await redis.quit();
      logger.info('Cleanup complete. Exiting gracefully.');
      process.exit(0);
    });

    // Enforce shutdown timeout if in-flight requests hang
    setTimeout(() => {
      logger.error('Shutdown grace period exceeded. Forcing exit.');
      process.exit(1);
    }, 15000).unref();
  }

  process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
  process.on('SIGINT', () => gracefulShutdown('SIGINT'));
  ```
- **Exceptions**: None.

---

## Rule DEP-04: Two-Phase Database Migrations (Expand-Contract Pattern)
- **Rule**: Database schema modifications MUST be deployed using the Expand-Contract pattern to ensure zero-downtime compatibility. A migration MUST NOT drop or rename a column that the currently running version of the application code still references.
- **Reason**: During rolling updates, the old code version and the new code version run concurrently side-by-side for several minutes. Dropping a column immediately causes the old code pods to throw 500 errors.
- **Risk**: High-volume production downtime during rolling releases.
- **Correct Migration Sequence**:
  - **Phase 1 (Expand)**: Add the new column/table as optional or nullable. Deploy code version that writes to *both* old and new columns.
  - **Phase 2 (Backfill)**: Run background script to copy old data to new column.
  - **Phase 3 (Contract)**: Deploy code version that reads/writes *only* the new column.
  - **Phase 4 (Cleanup)**: Drop the old column in a subsequent migration.
- **Exceptions**: Pre-production greenfield projects with zero live users.
