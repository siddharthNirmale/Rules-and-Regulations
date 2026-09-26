# Authorization & Access Control Rules

> **Standard**: Server-Side Authorization, IDOR/BOLA Prevention, Tenant Isolation, and Least Privilege.

---

## The Authorization Invariant
Authentication verifies **identity** ("Who are you?"). Authorization verifies **permission** ("Are you permitted to perform this specific action on this specific object right now?"). The server MUST enforce authorization on every single request, regardless of client UI state.

---

## Rule AZ-01: Never Rely on Client-Side UI Controls for Authorization [MUST]
- **Rule**: Hiding, disabling, or omitting a button, link, or navigation item in the frontend MUST NOT be considered a security control. Every protected action MUST be verified on the server.
- **Reason**: Any user can inspect network traffic, forge HTTP requests using `curl`, or modify client JavaScript state in browser developer tools to trigger hidden endpoints.
- **Risk**: Unauthorized users performing administrative actions or accessing restricted features.
- **Correct Approach**:
  ```typescript
  // ❌ VULNERABLE: Only checking role on the frontend; backend route is unprotected!
  app.delete('/api/v1/users/:id', async (req, res) => {
    await db.users.delete({ where: { id: req.params.id } });
    res.status(204).send();
  });

  // ✅ CORRECT: Backend middleware strictly enforces authorization
  app.delete(
    '/api/v1/users/:id',
    authenticateUser,
    requireRole(['ADMIN', 'SUPER_ADMIN']),
    async (req, res) => {
      await userService.deleteUser(req.params.id);
      res.status(204).send();
    }
  );
  ```
- **Exceptions**: None.

---

## Rule AZ-02: Prevent Insecure Direct Object References (IDOR / BOLA) [MUST]
- **Rule**: When retrieving, updating, or deleting a resource by ID, the query MUST explicitly filter by the authenticated user's ID or organization/tenant ID.
- **Reason**: Without resource ownership checks, an attacker who is authenticated as User A can view or modify User B's private invoices, profiles, or documents simply by changing `?id=123` to `?id=124`.
- **Risk**: Mass unauthorized data exfiltration across all accounts in the database (OWASP API Top 10 #1 vulnerability).
- **Correct Approach**:
  ```typescript
  // ❌ S1 CRITICAL VULNERABILITY (IDOR): Any user can view any invoice!
  app.get('/api/invoices/:id', authenticateUser, async (req, res) => {
    const invoice = await db.invoices.findUnique({
      where: { id: req.params.id }
    });
    res.json(invoice);
  });

  // ✅ CORRECT: Scoped directly to the authenticated organization/user
  app.get('/api/invoices/:id', authenticateUser, async (req, res) => {
    const invoice = await db.invoices.findFirst({
      where: {
        id: req.params.id,
        organizationId: req.user.organizationId // Mandatory ownership filter
      }
    });

    if (!invoice) {
      // Return 404 to avoid leaking whether the invoice ID exists in other orgs
      return res.status(404).json({ error: 'Invoice not found' });
    }

    return res.json({ success: true, data: invoice });
  });
  ```
- **Exceptions**: Publicly published entities (e.g. public blog posts, open store products).

---

## Rule AZ-03: Multi-Tenant Data Isolation [CONTEXT-DEPENDENT]
- **Rule**: In multi-tenant systems, all database schemas or queries MUST enforce tenant isolation at the persistence level (e.g., Row-Level Security in PostgreSQL or tenant middleware that automatically injects `tenant_id` into all queries).
- **Applicability**: Mandatory for multi-tenant software sharing a single database across multiple corporate accounts.
- **Reason**: Relying on developers to remember to append `WHERE tenant_id = ?` to hundreds of queries inevitably leads to human omission.
- **Risk**: Cross-tenant data leaks and enterprise compliance violations.
- **Correct Approach**:
  ```sql
  -- PostgreSQL Row Level Security (RLS)
  ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

  CREATE POLICY tenant_isolation_policy ON documents
    AS RESTRICTIVE
    USING (tenant_id = CURRENT_SETTING('app.current_tenant_id')::UUID);
  ```
- **Exceptions**: Physically isolated single-tenant database architectures (database-per-tenant).

---

## Rule AZ-04: Least Privilege for Service Accounts and Database Users [MUST]
- **Rule**: The database user and service account credentials used by the web application MUST possess only the minimum permissions required for standard operations (e.g. `SELECT`, `INSERT`, `UPDATE`, `DELETE`). The application user MUST NOT have permissions to `DROP TABLE`, `CREATE DATABASE`, or `ALTER SCHEMA`.
- **Reason**: If a SQL injection vulnerability is discovered, limiting database user privileges prevents the attacker from dropping tables, creating new admin accounts, or reading database audit logs.
- **Risk**: Catastrophic database destruction or unauthorized schema corruption.
- **Correct Approach**:
  - Web runtime user: Grants only `DML` (`SELECT`, `INSERT`, `UPDATE`, `DELETE`).
  - Migration runner: Runs under a separate, elevated CI/CD pipeline user only during deployment.
- **Exceptions**: Local developer SQLite sandboxes.
