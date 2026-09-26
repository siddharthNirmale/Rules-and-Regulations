# Database Engineering & Persistence Rules

> **Scope**: Relational (PostgreSQL, MySQL) and Document/NoSQL (MongoDB, DynamoDB) persistence standards.

---

## Rule DB-01: Absolute Parameterization of All Database Queries [MUST]
- **Rule**: SQL and NoSQL queries MUST use parameterized queries, prepared statements, or ORM parameter bindings exclusively. Direct string interpolation or concatenation of variables into query strings is an **S1 Critical Violation**.
- **Reason**: String interpolation permits SQL/NoSQL Injection, allowing untrusted input to break out of data literals into executable query syntax.
- **Risk**: Full database compromise, unauthorized administrative escalation, and deletion of all table data.
- **Correct Approach**:
  ```typescript
  // ❌ S1 CRITICAL VIOLATION: SQL Injection
  const user = await db.query(`SELECT * FROM users WHERE email = '${email}'`);

  // ✅ CORRECT: Parameterized query binding
  const user = await db.query(
    'SELECT id, email, role, password_hash FROM users WHERE email = $1',
    [email]
  );
  ```
- **Exceptions**: None. Column names or table identifiers that cannot be parameterized MUST be checked against a strict, static allowlist.

---

## Rule DB-02: Zero N+1 Query Loops [MUST]
- **Rule**: Relational queries MUST NOT fetch child records inside application-level loops over parent records. Use SQL `JOIN`s, eager loading (`include`), or batched data-loaders.
- **Reason**: Looping over 100 parent records to query their child records triggers 101 distinct database round-trips (1 + N).
- **Risk**: Severe latency spikes, database CPU exhaustion, and connection pool starvation.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: 1 + N queries
  const users = await db.query('SELECT id, name FROM users LIMIT 50');
  for (const user of users.rows) {
    user.posts = (await db.query('SELECT title FROM posts WHERE user_id = $1', [user.id])).rows;
  }

  // ✅ CORRECT: Single joined query with JSON aggregation
  const result = await db.query(`
    SELECT u.id, u.name,
      COALESCE(json_agg(json_build_object('title', p.title)) FILTER (WHERE p.id IS NOT NULL), '[]') AS posts
    FROM users u
    LEFT JOIN posts p ON p.user_id = u.id
    GROUP BY u.id, u.name
    LIMIT 50
  `);
  ```
- **Exceptions**: Distributed microservices where parent and child entities live in physically separate database engines.

---

## Rule DB-03: Mandatory Indexing on Foreign Keys & Filter Predicates [MUST]
- **Rule**: Every foreign key column, frequently queried `WHERE` condition, and `ORDER BY` timestamp column MUST be backed by a corresponding database index.
- **Reason**: Without an index, the database engine must execute a Full Table Scan, reading millions of disk blocks sequentially.
- **Risk**: Queries that execute in 2ms on dev tables with 100 rows degrade to 15 seconds on production tables with 1,000,000 rows.
- **Correct Approach**:
  ```sql
  -- Index foreign key to prevent slow joins
  CREATE INDEX idx_orders_customer_id ON orders(customer_id);

  -- Composite index for multi-column queries: WHERE customer_id = ? AND status = ?
  CREATE INDEX idx_orders_customer_status ON orders(customer_id, status);
  ```
- **Exceptions**: Low-cardinality boolean flag columns on tiny lookup tables under 1,000 rows.

---

## Rule DB-04: Forward-Only, Append-Only Migrations [MUST]
- **Rule**: Once a database migration has been committed and merged into the main development branch, it MUST NEVER be edited. All schema changes, additions, rollbacks, and data fixes MUST be applied via new, forward-only migration scripts.
- **Reason**: Modifying an existing migration creates irreconcilable schema drift between developer machines, staging clusters, and production environments.
- **Risk**: Failed CI deployments, corrupted production database schemas, and migration lock crashes.
- **Correct Approach**:
  ```bash
  # ❌ BAD: Editing 20240101_initial_schema.sql to add a column
  
  # ✅ CORRECT: Generate a new forward migration
  npm run migration:create add_phone_number_to_users
  ```
- **Exceptions**: Local sandbox development branches prior to opening a PR.

---

## Rule DB-05: Explicit Column Projections (Avoid `SELECT *`) [MUST]
- **Rule**: Production queries MUST explicitly enumerate the required columns rather than using wildcard `SELECT *`.
- **Reason**: `SELECT *` transfers unnecessary columns over the network, prevents index-only scans, and risks leaking newly added sensitive columns (e.g. `password_reset_token`) directly to API clients.
- **Risk**: Increased network overhead, slower query execution, and accidental data exposure.
- **Correct Approach**:
  ```typescript
  // ✅ CORRECT: Explicit column selection
  const user = await db.query(
    'SELECT id, username, email, created_at FROM users WHERE id = $1',
    [userId]
  );
  ```
- **Exceptions**: Ad-hoc debugging scripts in development or database introspection tools.
