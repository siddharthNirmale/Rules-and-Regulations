# Comprehensive Analysis of Common AI Coding Mistakes

> **Purpose**: A definitive technical autopsy of the 22 canonical failure patterns exhibited by AI coding agents across the entire software development lifecycle, with root-cause analysis, operational risks, prevention methods, enforceable rules, and concrete Bad vs. Good code demonstrations.

---

## Table of Contents
1. [Frontend Development](#1-frontend-development)
2. [Backend Development](#2-backend-development)
3. [UI/UX Implementation](#3-uiux-implementation)
4. [Database Design and Interaction](#4-database-design-and-interaction)
5. [API Development](#5-api-development)
6. [Authentication and Authorization](#6-authentication-and-authorization)
7. [Cybersecurity](#7-cybersecurity)
8. [Data Validation and Sanitization](#8-data-validation-and-sanitization)
9. [Error Handling](#9-error-handling)
10. [Performance Optimization](#10-performance-optimization)
11. [Accessibility (a11y)](#11-accessibility-a11y)
12. [SEO (Search Engine Optimization)](#12-seo-search-engine-optimization)
13. [Responsive Design](#13-responsive-design)
14. [State Management](#14-state-management)
15. [Code Architecture](#15-code-architecture)
16. [Dependency Management](#16-dependency-management)
17. [Testing](#17-testing)
18. [DevOps and Deployment](#18-devops-and-deployment)
19. [Configuration and Environment Variables](#19-configuration-and-environment-variables)
20. [Logging and Monitoring](#20-logging-and-monitoring)
21. [Documentation](#21-documentation)
22. [Refactoring and Maintenance](#22-refactoring-and-maintenance)

---

## 1. Frontend Development

### 1.1 Common Mistakes AI Makes
- Generating div-soup instead of semantic HTML5 tags (`<div onClick={...}>` instead of `<button>`).
- Excessive JavaScript for behaviors that standard CSS handles natively (custom JS scroll, manual hover dropdowns, custom modal backdrops).
- Storing derived state in duplicate `useState` hooks, causing desynchronization bugs.
- Omitting cleanup functions in `useEffect` for timers, event listeners, or WebSockets.
- Direct DOM manipulation (`document.getElementById`) inside declarative framework components.

### 1.2 Why Those Mistakes Are Problematic
- Div-based clickables break keyboard navigation (Tab/Enter/Space), screen readers, and touch target standards.
- Excessive JS bloats client bundles, delays First Contentful Paint (FCP) and Time to Interactive (TTI), and increases battery drain on mobile devices.
- Unsynced duplicate state leads to ghost UI states where the UI renders contradictory information.
- Missing cleanup causes memory leaks that crash browser tabs over extended user sessions.

### 1.3 How They Should Be Prevented
- Always default to semantic HTML5 elements before adding any JavaScript or styling.
- Compute derived values on the fly during render; do not sync them via `useEffect` + `setState`.
- Enforce that every subscription or listener returned from an effect has a corresponding teardown.

### 1.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** use native semantic HTML elements for interactive controls, **MUST NOT** replicate native browser capabilities with JavaScript, and **MUST** provide immediate teardown for any resource allocated in component lifecycles.

### 1.5 Bad vs. Correct Approach
```tsx
// ❌ BAD: Div clickable, derived state synced via useEffect, missing cleanup
function UserSearch() {
  const [query, setQuery] = useState('');
  const [items, setItems] = useState<string[]>([]);
  const [filteredItems, setFilteredItems] = useState<string[]>([]);

  useEffect(() => {
    // Unnecessary state synchronization
    setFilteredItems(items.filter(i => i.toLowerCase().includes(query.toLowerCase())));
  }, [query, items]);

  useEffect(() => {
    // Memory leak: listener never removed
    window.addEventListener('resize', () => console.log(window.innerWidth));
  }, []);

  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <div className="btn" onClick={() => alert('Search clicked!')}>Search</div>
      {filteredItems.map(item => <div key={item}>{item}</div>)}
    </div>
  );
}

// ✅ CORRECT: Semantic button, derived value inlined, listener cleaned up
function UserSearch({ items }: { items: string[] }) {
  const [query, setQuery] = useState('');

  // Derived state computed in-line (zero synchronization lag)
  const filteredItems = useMemo(
    () => items.filter(i => i.toLowerCase().includes(query.toLowerCase())),
    [items, query]
  );

  useEffect(() => {
    const handleResize = () => console.log(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize); // Teardown
  }, []);

  return (
    <section aria-label="User search">
      <label htmlFor="user-query" className="sr-only">Search users</label>
      <input
        id="user-query"
        type="search"
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Filter users..."
      />
      <button type="button" onClick={() => alert('Search clicked!')}>
        Search
      </button>
      <ul role="list">
        {filteredItems.map(item => <li key={item}>{item}</li>)}
      </ul>
    </section>
  );
}
```

---

## 2. Backend Development

### 2.1 Common Mistakes AI Makes
- Mixing presentation, routing, business logic, and database operations inside single controller functions.
- Blocking the asynchronous event loop with heavy synchronous CPU tasks (e.g., synchronous cryptographic hashing, massive JSON parsing).
- Failing to handle concurrency, race conditions, or distributed consistency (e.g., read-then-write without row locks or transactions).
- Swallowing unhandled promise rejections or failing to attach top-level process crash handlers.

### 2.2 Why Those Mistakes Are Problematic
- Monolithic controllers make unit testing impossible, create tight coupling, and prevent code reuse.
- Blocking the event loop causes latency spikes for all concurrent users, causing timeouts and denial of service.
- Unhandled race conditions allow double-spending, inventory overselling, or corrupted state.

### 2.3 How They Should Be Prevented
- Enforce strict 3-tier layering: Controller (HTTP) -> Service (Domain/Business logic) -> Repository/DAO (Persistence).
- Offload heavy compute tasks to worker threads or background job queues.
- Guard state mutations inside atomic database transactions with proper isolation levels.

### 2.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** isolate HTTP handling, business logic, and data persistence into distinct decoupled layers, and **MUST** execute mutating multi-step data operations within atomic transactions.

### 2.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Business logic, DB mutation, and response generation in one untransacted handler
app.post('/transfer', async (req, res) => {
  const { fromId, toId, amount } = req.body;
  const sender = await db.query('SELECT balance FROM accounts WHERE id = ' + fromId);
  if (sender.rows[0].balance >= amount) {
    // Race condition: concurrent transfer can overdraft
    await db.query(`UPDATE accounts SET balance = balance - ${amount} WHERE id = ${fromId}`);
    await db.query(`UPDATE accounts SET balance = balance + ${amount} WHERE id = ${toId}`);
    res.json({ success: true });
  } else {
    res.status(400).send('Insufficient funds');
  }
});

// ✅ CORRECT: Layered architecture, parameterized queries, atomic ACID transaction
export class AccountService {
  constructor(private readonly accountRepo: AccountRepository) {}

  async transfer(fromId: string, toId: string, amount: number): Promise<void> {
    if (amount <= 0) throw new ValidationError('Amount must be positive');
    
    await this.accountRepo.executeTransaction(async (trx) => {
      // FOR UPDATE locks rows to prevent concurrent balance overdrafts
      const sender = await trx.getAccountForUpdate(fromId);
      if (!sender || sender.balance < amount) {
        throw new InsufficientFundsError();
      }
      await trx.decrementBalance(fromId, amount);
      await trx.incrementBalance(toId, amount);
    });
  }
}
```

---

## 3. UI/UX Implementation

### 3.1 Common Mistakes AI Makes
- Implementing only the "happy path" UI: omitting loading spinners, skeleton states, empty states, and error retry prompts.
- Destructive actions without confirmation modals or undo mechanisms.
- Layout shifts (Cumulative Layout Shift) caused by un-dimensioned images or dynamically injected banners.
- Inconsistent color scales, mismatched margins, and chaotic typography violating design systems.

### 3.2 Why Those Mistakes Are Problematic
- Missing loading/error states leave users wondering if their action registered or if the app froze.
- Accidental clicks on destructive buttons cause irreversible data loss.
- CLS frustrates users and degrades SEO search rankings.

### 3.3 How They Should Be Prevented
- Every data-fetching UI component must handle four distinct states: Initial/Idle, Loading, Error, and Empty.
- Destructive operations must require two distinct intentional steps (modal confirmation, double-click, or temporary undo banner).
- Reserve layout dimensions using aspect-ratio or skeleton placeholders.

### 3.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** explicitly implement Loading, Error, Empty, and Success states for every dynamic view, and **MUST** require explicit confirmation for destructive actions.

### 3.5 Bad vs. Correct Approach
```tsx
// ❌ BAD: No loading indicator, crashes on error, deletes immediately without confirm
function UserList() {
  const { data } = useUsers();
  return (
    <div>
      {data.map(u => (
        <div key={u.id}>
          {u.name}
          <button onClick={() => deleteUser(u.id)}>Delete</button>
        </div>
      ))}
    </div>
  );
}

// ✅ CORRECT: Exhaustive state handling, layout preservation, destructive confirmation
function UserList() {
  const { data: users, isLoading, error, refetch } = useUsers();
  const [pendingDeleteId, setPendingDeleteId] = useState<string | null>(null);

  if (isLoading) return <UserListSkeleton count={5} />;
  if (error) return <ErrorState message="Failed to load users" onRetry={refetch} />;
  if (!users || users.length === 0) return <EmptyState title="No users found" actionLabel="Invite User" />;

  return (
    <>
      <ul role="list" className="divide-y divide-slate-200">
        {users.map(user => (
          <li key={user.id} className="flex justify-between items-center py-3">
            <span className="font-medium text-slate-800">{user.name}</span>
            <button
              type="button"
              className="text-red-600 hover:text-red-800 text-sm font-semibold"
              onClick={() => setPendingDeleteId(user.id)}
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
      {pendingDeleteId && (
        <ConfirmDialog
          title="Delete Account"
          description="Are you sure you want to delete this user? This action cannot be undone."
          confirmLabel="Delete"
          isDestructive
          onConfirm={async () => {
            await deleteUser(pendingDeleteId);
            setPendingDeleteId(null);
          }}
          onCancel={() => setPendingDeleteId(null)}
        />
      )}
    </>
  );
}
```

---

## 4. Database Design and Interaction

### 4.1 Common Mistakes AI Makes
- Concatenating raw user inputs into SQL strings (SQL Injection).
- Introducing N+1 queries by fetching relationships inside application loops.
- Omitting indexes on foreign keys, `WHERE` search columns, and `ORDER BY` timestamps.
- Creating unstructured schema migrations or modifying existing migration files that have already run in production.
- Using `SELECT *` across wide tables, pulling MBs of unused binary data or sensitive columns into memory.

### 4.2 Why Those Mistakes Are Problematic
- SQL injection allows total database exfiltration and server destruction.
- N+1 queries bring high-traffic databases to a halt through connection pool starvation.
- Unindexed queries degrade from 5ms index seeks to 10-second table scans as tables scale.
- Altering past migrations creates schema desynchronization across staging and production environments.

### 4.3 How They Should Be Prevented
- Mandate parameterized queries / prepared statements for every query.
- Use eager loading (`JOIN`, `include`) or dataloaders for relational queries.
- Add indexes to all foreign keys and frequently queried filter/sort predicates.
- Enforce append-only database migrations.

### 4.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** use parameterized queries exclusively, **MUST NOT** emit N+1 query patterns, **MUST** index queried foreign keys, and **MUST** write forward-only database migrations.

### 4.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: SQL injection, SELECT *, N+1 query loop
async function getAuthorSummaries(authorName: string) {
  // Vulnerable to SQL injection!
  const authors = await db.query(`SELECT * FROM authors WHERE name = '${authorName}'`);
  
  for (const author of authors.rows) {
    // N+1 query: 1 query for authors + N queries for books!
    const books = await db.query(`SELECT * FROM books WHERE author_id = ${author.id}`);
    author.books = books.rows;
  }
  return authors.rows;
}

// ✅ CORRECT: Parameterized, explicit columns, single joined query
async function getAuthorSummaries(authorName: string): Promise<AuthorSummary[]> {
  const query = `
    SELECT 
      a.id, a.name, a.email,
      COALESCE(
        json_agg(
          json_build_object('id', b.id, 'title', b.title, 'published_year', b.published_year)
        ) FILTER (WHERE b.id IS NOT NULL), '[]'
      ) AS books
    FROM authors a
    LEFT JOIN books b ON b.author_id = a.id
    WHERE a.name = $1
    GROUP BY a.id, a.name, a.email
  `;
  const result = await db.query(query, [authorName]);
  return result.rows;
}
```

---

## 5. API Development

### 5.1 Common Mistakes AI Makes
- Returning HTTP 200 OK for business errors (e.g., `{ success: false, error: "Unauthorized" }` with status 200).
- Unbounded collection endpoints returning the entire database table without pagination.
- Inconsistent payload shapes (sometimes `{ data: [...] }`, sometimes `[...]`, sometimes `{ items: [...] }`).
- Ignoring HTTP method semantics (using GET for state-mutating deletions, POST for read-only fetches).
- Omitting request timeout handling and idempotency keys on payment or critical mutation endpoints.

### 5.2 Why Those Mistakes Are Problematic
- HTTP 200 for errors fools API gateways, client libraries, and monitoring systems into recording failures as healthy requests.
- Unbounded endpoints cause memory exhaustion and server crashes once collections grow to thousands of records.
- Inconsistent payloads break client type generators and require redundant defensive code.

### 5.3 How They Should Be Prevented
- Adhere strictly to RFC HTTP status code semantics (400 for validation, 401 for unauth, 403 for forbidden, 404 for missing, 500 for server error).
- Require cursor or limit/offset pagination on every collection endpoint with a enforced maximum limit.
- Standardize on an API response contract (e.g., JSON:API or a typed `{ success, data, error, meta }` envelope).

### 5.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** return semantically accurate HTTP status codes, **MUST** enforce pagination with strict maximum limits on all collection endpoints, and **MUST** maintain consistent API response envelopes.

### 5.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Returns 200 for error, unbounded fetch, GET mutation
app.get('/api/deleteUser', async (req, res) => {
  if (!req.query.id) {
    return res.status(200).json({ error: 'Missing ID' }); // Status 200 for error!
  }
  await db.deleteUser(req.query.id);
  res.json({ message: 'Deleted' });
});

app.get('/api/users', async (req, res) => {
  const users = await db.query('SELECT * FROM users'); // May return 500,000 rows!
  res.json(users);
});

// ✅ CORRECT: Semantic verbs, appropriate status codes, enforced pagination envelope
app.delete('/api/v1/users/:id', authenticate, authorize('admin'), async (req, res) => {
  await userService.deleteUser(req.params.id);
  return res.status(204).send(); // 204 No Content
});

app.get('/api/v1/users', authenticate, async (req, res) => {
  const page = Math.max(1, parseInt(req.query.page as string) || 1);
  const limit = Math.min(100, Math.max(1, parseInt(req.query.limit as string) || 20)); // Enforced cap

  const { items, total } = await userService.listUsers({ page, limit });
  return res.status(200).json({
    success: true,
    data: items,
    meta: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit)
    }
  });
});
```

---

## 6. Authentication and Authorization

### 6.1 Common Mistakes AI Makes
- Storing authentication tokens (JWTs) in browser `localStorage` or `sessionStorage` (vulnerable to XSS extraction).
- Confusing authentication ("who you are") with authorization ("what you can do").
- Insecure Direct Object References (IDOR / BOLA): checking if a user is logged in, but not checking if the requested resource belongs to that user (`SELECT * FROM invoices WHERE id = :id` instead of `WHERE id = :id AND user_id = :current_user`).
- Using weak, obsolete password hashing algorithms (MD5, SHA-1, SHA-256) instead of adaptive work-factor schemes (Argon2id, bcrypt).
- Missing cryptographic signature verification on JWT tokens or accepting the `alg: "none"` header.

### 6.2 Why Those Mistakes Are Problematic
- LocalStorage tokens can be exfiltrated by any third-party script injected via XSS.
- IDOR vulnerabilities allow any authenticated user to view or delete another user's private data simply by altering an ID in the URL.
- Fast hash functions (SHA-256) can be cracked at billions of hashes per second using standard consumer GPUs.

### 6.3 How They Should Be Prevented
- Store session tokens in `HttpOnly`, `Secure`, `SameSite=Lax/Strict` cookies.
- Implement fine-grained resource-level ownership validation on every data access query.
- Use Argon2id (or bcrypt with work factor >= 12) for password hashing.
- Verify JWT signatures strictly against known public keys or secrets, rejecting unknown algorithms.

### 6.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** store session tokens in `HttpOnly`, `Secure` cookies, **MUST** verify resource ownership (IDOR prevention) on every data-mutating or fetching route, and **MUST** hash passwords using Argon2id or bcrypt.

### 6.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Token stored in localStorage, IDOR vulnerability, SHA-256 hashing
// Client-side:
localStorage.setItem('authToken', token);

// Server-side:
app.get('/api/documents/:id', async (req, res) => {
  // Vulnerable to IDOR: Any logged-in user can read ANY user's documents!
  const doc = await db.query('SELECT * FROM documents WHERE id = $1', [req.params.id]);
  res.json(doc.rows[0]);
});

async function hashPassword(password: string) {
  // BROKEN: Fast hashing algorithm is trivial to crack
  return crypto.createHash('sha256').update(password).digest('hex');
}

// ✅ CORRECT: HttpOnly cookie, resource authorization check, Argon2id
// Server-side set cookie:
res.cookie('session_id', sessionId, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'lax',
  maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
});

// Resource ownership check:
app.get('/api/v1/documents/:id', authenticateUser, async (req, res) => {
  const doc = await db.query(
    'SELECT id, title, content, created_at FROM documents WHERE id = $1 AND organization_id = $2',
    [req.params.id, req.user.organizationId] // Scope to tenant/user
  );
  if (doc.rows.length === 0) {
    return res.status(404).json({ error: 'Document not found' }); // Don't leak existence
  }
  return res.json({ success: true, data: doc.rows[0] });
});

import argon2 from 'argon2';
async function hashPassword(password: string): Promise<string> {
  return argon2.hash(password, { type: argon2.argon2id, memoryCost: 65536, timeCost: 3 });
}
```

---

## 7. Cybersecurity

### 7.1 Common Mistakes AI Makes
- Rendering unescaped user HTML via `dangerouslySetInnerHTML`, `v-html`, or `innerHTML` (Cross-Site Scripting).
- Concatenating user inputs into operating system shells (`child_process.exec(userCmd)`).
- Allowing Server-Side Request Forgery (SSRF) by fetching arbitrary user-supplied URLs without IP/hostname allowlisting.
- Omitting modern security headers (`Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`).
- Path traversal vulnerabilities allowing clients to download arbitrary files (`res.sendFile('/uploads/' + req.query.filename)`).

### 7.2 Why Those Mistakes Are Problematic
- XSS allows session hijacking, keylogging, and defacement within the user's browser.
- Command injection allows arbitrary remote code execution (RCE) on the host server.
- SSRF allows attackers to probe internal cloud metadata services (e.g., `http://169.254.169.254/latest/meta-data/`) to steal IAM credentials.
- Path traversal leaks system configuration files like `/etc/passwd` or `.env`.

### 7.3 How They Should Be Prevented
- Always sanitize HTML using an audited library like DOMPurify, or avoid raw HTML injection altogether.
- Never invoke OS shells with string concatenation; use `execFile` with argument arrays.
- Validate and resolve external URLs, strictly forbidding private IP ranges (RFC 1918, link-local, loopback).
- Configure strict HTTP security headers at the web server or reverse proxy level.
- Validate file paths using base directory resolution and reject filenames containing path traversal sequences (`..`).

### 7.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** sanitize any dynamic HTML with DOMPurify, **MUST NOT** execute concatenated shell strings, **MUST** block private network ranges on server-side HTTP fetches (SSRF defense), and **MUST** sanitize all file paths against path traversal.

### 7.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Command injection, SSRF, Path traversal
app.get('/convert', (req, res) => {
  // Command injection: attacker inputs `image.png; rm -rf /`
  exec(`convert ${req.query.file} out.jpg`); 
});

app.post('/fetch-preview', async (req, res) => {
  // SSRF: attacker targets http://169.254.169.254/latest/meta-data/
  const response = await fetch(req.body.url);
  res.send(await response.text());
});

app.get('/download', (req, res) => {
  // Path traversal: filename = "../../.env"
  res.sendFile(path.join(__dirname, 'files', req.query.filename));
});

// ✅ CORRECT: Safe argument array, SSRF protection, strict path normalization
import { execFile } from 'child_process';
import ipaddr from 'ipaddr.js';

app.get('/convert', (req, res) => {
  const safeFilename = path.basename(req.query.file as string);
  // Safely passes arguments without invoking a shell
  execFile('convert', [safeFilename, 'out.jpg'], (err) => {
    if (err) return res.status(500).send('Conversion error');
    res.download('out.jpg');
  });
});

app.post('/fetch-preview', async (req, res) => {
  const targetUrl = new URL(req.body.url);
  if (!['http:', 'https:'].includes(targetUrl.protocol)) {
    return res.status(400).send('Invalid protocol');
  }

  const dnsLookup = await dns.promises.lookup(targetUrl.hostname);
  const addr = ipaddr.parse(dnsLookup.address);
  // Block private/loopback/cloud metadata IP ranges
  if (addr.range() !== 'unicast' || addr.toString() === '169.254.169.254') {
    return res.status(403).send('Forbidden destination IP');
  }

  const response = await fetch(targetUrl.toString(), { redirect: 'error' });
  res.send(await response.text());
});

app.get('/download', (req, res) => {
  const safeFilename = path.basename(req.query.filename as string);
  const basePath = path.resolve(__dirname, 'safe_storage');
  const targetPath = path.resolve(basePath, safeFilename);

  if (!targetPath.startsWith(basePath)) {
    return res.status(403).send('Access denied');
  }
  res.sendFile(targetPath);
});
```

---

## 8. Data Validation and Sanitization

### 8.1 Common Mistakes AI Makes
- Relying exclusively on client-side form validation (e.g., HTML `required` attributes or React state) without server-side validation.
- Validating inputs using loose, permissive regular expressions that cause catastrophic backtracking (ReDoS).
- Accepting arbitrary unvalidated JSON keys into database updates (mass-assignment vulnerability).
- Missing type and bounds validation (e.g., accepting negative quantities, 100MB string payloads).

### 8.2 Why Those Mistakes Are Problematic
- Anyone can bypass client-side validation using curl, Postman, or automated scripts.
- Mass-assignment allows attackers to send `{ "isAdmin": true }` or `{ "balance": 999999 }` in a profile update payload.
- Uncapped string inputs exhaust server memory or crash database column length constraints.

### 8.3 How They Should Be Prevented
- Enforce schema-based validation (using Zod, Joi, Valibot, or Pydantic) on every incoming API request payload.
- Strip unknown or unpermitted fields using strict schema parsing (e.g., `zod.strict()` or picking allowed fields).
- Enforce minimum and maximum boundaries on all strings, numbers, and arrays.

### 8.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** validate and parse all incoming server requests using strict, bounded validation schemas, and **MUST NOT** perform mass assignment of request payloads directly to database models.

### 8.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: No schema validation, mass assignment, unconstrained input
app.put('/api/profile', async (req, res) => {
  // Attacker can pass { "role": "admin", "isVerified": true }
  await db.users.update({ where: { id: req.user.id }, data: req.body });
  res.json({ success: true });
});

// ✅ CORRECT: Strict schema parsing, bounded fields, whitelisted properties
import { z } from 'zod';

const UpdateProfileSchema = z.object({
  displayName: z.string().trim().min(1).max(50),
  bio: z.string().max(500).optional(),
  newsletterSubscribed: z.boolean()
}).strict(); // Disallows unexpected fields

app.put('/api/profile', authenticate, async (req, res) => {
  const parseResult = UpdateProfileSchema.safeParse(req.body);
  if (!parseResult.success) {
    return res.status(400).json({
      success: false,
      errors: parseResult.error.flatten().fieldErrors
    });
  }

  // Only validated, safe fields are persisted
  const updatedUser = await db.users.update({
    where: { id: req.user.id },
    data: parseResult.data
  });
  return res.json({ success: true, data: updatedUser });
});
```

---

## 9. Error Handling

### 9.1 Common Mistakes AI Makes
- Using empty `catch {}` blocks that swallow errors silently.
- Leaking raw database errors, stack traces, and internal server paths in production API error responses.
- Catching errors only to re-throw them without context or original cause chains.
- Failing to distinguish between expected operational errors (e.g., invalid input, item not found) and programmer bugs (e.g., null pointer, syntax error).

### 9.2 Why Those Mistakes Are Problematic
- Swallowed errors lead to silent data corruption, broken workflows, and untraceable production failures.
- Leaked stack traces reveal internal file structures, database table names, and library versions to attackers.
- Conflating operational errors with system crashes results in inappropriate process restarts or 500 status codes for 400 bad requests.

### 9.3 How They Should Be Prevented
- Implement centralized error-handling middleware that maps custom domain error classes to HTTP status codes.
- Log complete error objects (including message, stack trace, and correlation ID) internally to structured logging systems.
- Return clean, sanitized JSON error envelopes to public clients with zero internal system details in production.

### 9.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST NOT** swallow errors silently, **MUST NOT** expose internal stack traces or database errors to public clients, and **MUST** handle errors via centralized error middleware with distinct operational error hierarchies.

### 9.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Silent swallowing, leaking internal stack trace to client
app.post('/process-order', async (req, res) => {
  try {
    await chargeCard(req.body);
  } catch (e) {
    // Swallowed! User doesn't know payment failed!
  }

  try {
    await fulfillOrder(req.body);
  } catch (err) {
    // Leaks database credentials and file paths in stack trace!
    res.status(500).json({ error: err.stack, details: err });
  }
});

// ✅ CORRECT: Typed domain errors, centralized handling, sanitized client envelope
export class DomainError extends Error {
  constructor(message: string, public readonly statusCode: number, public readonly code: string) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class PaymentFailedError extends DomainError {
  constructor(reason: string) {
    super(`Payment processing failed: ${reason}`, 402, 'PAYMENT_FAILED');
  }
}

// Global Express Error Middleware
function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  const correlationId = req.headers['x-request-id'] || crypto.randomUUID();

  // Log full context internally for engineers
  logger.error('Unhandled request exception', {
    correlationId,
    error: err.message,
    stack: err.stack,
    path: req.path
  });

  if (err instanceof DomainError) {
    return res.status(err.statusCode).json({
      success: false,
      error: { code: err.code, message: err.message, correlationId }
    });
  }

  // Generic sanitized fallback for unexpected programmer errors
  return res.status(500).json({
    success: false,
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: 'An unexpected error occurred. Please contact support.',
      correlationId
    }
  });
}
```

---

## 10. Performance Optimization

### 10.1 Common Mistakes AI Makes
- Premature optimization: introducing complex caching layers, memoization, and web workers before measuring bottlenecks.
- Re-rendering entire component trees due to creating inline functions or objects inside parent components without profiling.
- Loading massive full-size images without modern formats (`webp`, `avif`), responsive `srcset`, or lazy-loading attributes.
- Synchronous file I/O or unbounded memory allocations in server handlers.

### 10.2 Why Those Mistakes Are Problematic
- Premature caching leads to cache invalidation bugs (one of the hardest problems in computer science).
- Unoptimized images consume multi-megabytes of cellular bandwidth and degrade Core Web Vitals (LCP).
- Unbounded memory allocations cause Node.js out-of-memory crashes (`heap out of memory`).

### 10.3 How They Should Be Prevented
- Optimize based on measurement and profiling, not speculation.
- Always use responsive images with `loading="lazy"`, proper `aspect-ratio`, and modern image formats.
- Stream large files and datasets instead of reading entire files into memory buffers.

### 10.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** prioritize measurement-backed optimizations, **MUST** apply lazy-loading and dimension attributes to media, and **MUST** use streams for large file transfers.

### 10.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Loads entire 2GB file into memory buffer before sending
app.get('/download-report', async (req, res) => {
  const fileBuffer = await fs.promises.readFile('/var/reports/annual-dump.csv');
  res.setHeader('Content-Type', 'text/csv');
  res.send(fileBuffer); // Crashes server if multiple users request concurrently!
});

// ✅ CORRECT: Streams file chunk by chunk with minimal memory footprint
import fs from 'fs';
app.get('/download-report', (req, res) => {
  const filePath = '/var/reports/annual-dump.csv';
  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Disposition', 'attachment; filename="annual-dump.csv"');
  
  const stream = fs.createReadStream(filePath);
  stream.on('error', (err) => res.status(500).send('Streaming error'));
  stream.pipe(res);
});
```

---

## 11. Accessibility (a11y)

### 11.1 Common Mistakes AI Makes
- Using non-interactive HTML elements (`<div>`, `<span>`) for clickable triggers without `role`, `tabIndex`, or keyboard listeners.
- Omitting `alt` text on `<img>` elements, or using placeholder text like `"image"` or `"icon"`.
- Removing the browser focus outline (`outline: none` or `outline: 0`) without providing an accessible alternative.
- Using color alone to convey crucial status information (e.g., green for success, red for error without icons or text).
- Missing form labels or using placeholder attributes as the sole input label.

### 11.2 Why Those Mistakes Are Problematic
- Millions of assistive technology users (screen readers, braille displays, switch devices) are completely blocked from using the application.
- Violates international accessibility laws (ADA, European Accessibility Act, Section 508), creating severe legal liabilities.
- Users relying on keyboard navigation cannot see where their focus cursor is when outlines are stripped.

### 11.3 How They Should Be Prevented
- Always use native `<button>`, `<a>`, `<input>`, `<select>` elements.
- Ensure all forms have explicit `<label htmlFor="...">` associations.
- Provide high-contrast focus rings using `:focus-visible`.
- Guarantee WCAG 2.2 AA color contrast ratios (minimum 4.5:1 for normal text, 3:1 for large text).

### 11.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** comply with WCAG 2.2 AA standards, **MUST NOT** strip `:focus-visible` outlines without accessible replacements, **MUST** associate explicit labels with all form controls, and **MUST NOT** convey information using color alone.

### 11.5 Bad vs. Correct Approach
```tsx
// ❌ BAD: Div button, placeholder as label, focus outline stripped, color-only status
function LoginForm() {
  return (
    <div>
      <input type="email" placeholder="Enter email" style={{ outline: 'none' }} />
      <div 
        style={{ backgroundColor: 'blue', color: 'white', padding: '10px' }}
        onClick={() => console.log('Submit')}
      >
        Sign In
      </div>
      <span style={{ color: 'red' }}>Account Locked</span>
    </div>
  );
}

// ✅ CORRECT: Semantic form, explicit label, visible focus ring, multi-modal status
function LoginForm() {
  return (
    <form onSubmit={(e) => { e.preventDefault(); /* handle submit */ }}>
      <div className="flex flex-col gap-1">
        <label htmlFor="user-email" className="text-sm font-medium text-slate-700">
          Email Address
        </label>
        <input
          id="user-email"
          type="email"
          required
          className="rounded border border-slate-300 px-3 py-2 focus-visible:outline-2 focus-visible:outline-blue-600"
        />
      </div>

      <button
        type="submit"
        className="mt-4 rounded bg-blue-600 px-4 py-2 font-semibold text-white hover:bg-blue-700 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600"
      >
        Sign In
      </button>

      <div role="alert" className="mt-3 flex items-center gap-2 text-red-700">
        <AlertTriangleIcon aria-hidden="true" className="w-5 h-5" />
        <span className="text-sm font-medium">Account Locked: Contact support to reset.</span>
      </div>
    </form>
  );
}
```

---

## 12. SEO (Search Engine Optimization)

### 12.1 Common Mistakes AI Makes
- Rendering all metadata and content exclusively on the client without SSR/SSG on public pages.
- Omitting unique `<title>` and `<meta name="description">` tags on individual route pages.
- Multiple `<h1>` tags on a single page or skipping heading hierarchy levels (`<h1>` straight to `<h4>`).
- Broken OpenGraph and Twitter Card social preview tags.
- Missing canonical URLs, causing duplicate content penalties.

### 12.2 Why Those Mistakes Are Problematic
- Search engine web crawlers cannot index dynamic JavaScript content properly, reducing discoverability.
- Poor heading hierarchy degrades search engine topical parsing and screen reader navigation.
- Missing social tags result in ugly or broken cards when users share links on social platforms.

### 12.3 How They Should Be Prevented
- Implement server-side rendering (SSR) or static generation (SSG) for all publicly discoverable content.
- Dynamically generate unique title, description, and canonical tags for every route.
- Enforce strict hierarchical heading structure: exactly one `<h1>` per page, followed by `<h2>`, `<h3>`.

### 12.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** provide unique title and description metadata per page, **MUST** enforce a single `<h1>` with sequential heading levels, and **MUST** include canonical and OpenGraph tags on public routes.

### 12.5 Bad vs. Correct Approach
```html
<!-- ❌ BAD: Generic title, missing meta description, multiple h1s, no canonical tag -->
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <h1>Company Name</h1>
    <p>Welcome</p>
    <h1>Our Products</h1> <!-- Disallowed: Multiple h1 tags -->
    <h4>Widget 1</h4>      <!-- Disallowed: Skipped h2 and h3 -->
  </body>
</html>

<!-- ✅ CORRECT: Unique SEO metadata, OpenGraph tags, canonical link, semantic headings -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Enterprise Cloud Analytics Platform | CloudPulse</title>
    <meta name="description" content="Monitor infrastructure, track microservice latency, and prevent downtime with CloudPulse's automated telemetry analytics platform.">
    <link rel="canonical" href="https://cloudpulse.io/platform/analytics">
    
    <!-- OpenGraph Metadata -->
    <meta property="og:title" content="Enterprise Cloud Analytics Platform | CloudPulse">
    <meta property="og:description" content="Monitor infrastructure and prevent downtime with automated telemetry.">
    <meta property="og:image" content="https://cloudpulse.io/images/og-analytics.jpg">
    <meta property="og:type" content="website">
    <meta name="twitter:card" content="summary_large_image">
  </head>
  <body>
    <header>
      <nav aria-label="Main Navigation">...</nav>
    </header>
    <main>
      <h1>Cloud Telemetry Analytics</h1>
      <section>
        <h2>Real-Time Latency Monitoring</h2>
        <section>
          <h3>Microservice Tracing</h3>
        </section>
      </section>
    </main>
  </body>
</html>
```

---

## 13. Responsive Design

### 13.1 Common Mistakes AI Makes
- Using fixed-width containers (`width: 1200px`) that overflow horizontally on mobile screens.
- Setting font sizes or margins exclusively in pixels instead of relative units (`rem`, `ch`, `em`, `%`).
- Designing mobile touch targets smaller than 44x44 CSS pixels.
- Using desktop hover states as the only mechanism to reveal essential controls.
- Missing the essential mobile viewport meta tag (`<meta name="viewport" content="width=device-width, initial-scale=1">`).

### 13.2 Why Those Mistakes Are Problematic
- Horizontal scrolling on mobile devices ruins usability and causes users to bounce immediately.
- Hover-only triggers are completely inaccessible on touchscreens (smartphones, tablets).
- Tiny buttons result in "fat-finger" misclicks, creating frustration and ordering errors.

### 13.3 How They Should Be Prevented
- Follow mobile-first design: write base styles for small screens, then enhance using min-width media queries.
- Use fluid layouts with CSS Grid, Flexbox, and `clamp()`.
- Ensure all interactive elements satisfy minimum 44x44px touch target footprints.

### 13.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** design mobile-first using fluid layouts, **MUST NOT** use fixed pixel widths on layout containers, **MUST** provide touch targets >= 44x44px, and **MUST NOT** hide functionality behind hover-only interactions.

### 13.5 Bad vs. Correct Approach
```css
/* ❌ BAD: Fixed desktop width, microscopic touch targets, hover-only visibility */
.container {
  width: 1280px; /* Overflows on mobile! */
}
.action-button {
  width: 20px;
  height: 20px; /* Too small for fingers! */
  display: none;
}
.card:hover .action-button {
  display: block; /* Impossible on touchscreen! */
}

/* ✅ CORRECT: Fluid mobile-first layout, clamp typography, 48px touch targets */
.container {
  width: 100%;
  max-width: 80rem; /* Fluid with maximum ceiling */
  margin-inline: auto;
  padding-inline: clamp(1rem, 5vw, 2.5rem);
}

.action-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 2.75rem; /* 44px minimum */
  min-height: 2.75rem; /* 44px minimum */
  padding: 0.5rem 1rem;
  touch-action: manipulation;
}
```

---

## 14. State Management

### 14.1 Common Mistakes AI Makes
- Storing server cache in global client state managers (Redux, Zustand) instead of dedicated caching libraries (TanStack Query, SWR).
- Mutating state directly (`state.items.push(newItem)`) in frameworks that rely on immutable state comparison.
- Prop-drilling state through 10 component layers instead of using composition or lightweight context.
- Storing transient UI state (e.g., modal open, tab active) in the URL or conversely failing to store shareable state (search filter, pagination) in the URL.

### 14.2 Why Those Mistakes Are Problematic
- Re-inventing server caching in Redux leads to stale data, complex deduplication logic, and manual invalidation bugs.
- Direct state mutation bypasses framework change detection, resulting in stale UI renders.
- Failing to sync search filters to URL search params prevents users from bookmarking or sharing links.

### 14.3 How They Should Be Prevented
- Clearly separate Server State (use React Query/SWR) from Client UI State (Zustand/Context).
- Treat state as strictly immutable; use functional updates or libraries like Immer.
- Store shareable view configuration (active tab, search filter, page number) in URL search parameters.

### 14.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** separate server cache from client UI state, **MUST** treat state as strictly immutable, and **MUST** persist shareable filter/pagination state into URL query parameters.

### 14.5 Bad vs. Correct Approach
```tsx
// ❌ BAD: Direct mutation, storing server state manually in global store without cache handling
// store.ts:
const useStore = create((set) => ({
  users: [],
  addUser: (user) => set((state) => {
    state.users.push(user); // MUTATION! Fails reactivity
    return state;
  })
}));

// ✅ CORRECT: TanStack Query for server state, URL search params for filters
import { useQuery } from '@tanstack/react-query';
import { useSearchParams } from 'react-router-dom';

function UsersDirectory() {
  const [searchParams, setSearchParams] = useSearchParams();
  const currentQuery = searchParams.get('q') || '';

  // Server state managed with automated caching, deduplication, and refetching
  const { data: users, isLoading } = useQuery({
    queryKey: ['users', currentQuery],
    queryFn: () => api.fetchUsers(currentQuery)
  });

  const handleSearchChange = (newQuery: string) => {
    // Shareable state synced to URL
    setSearchParams(prev => {
      if (newQuery) prev.set('q', newQuery);
      else prev.delete('q');
      return prev;
    });
  };

  return (
    <div>
      <SearchInput value={currentQuery} onChange={handleSearchChange} />
      {/* render list */}
    </div>
  );
}
```

---

## 15. Code Architecture

### 15.1 Common Mistakes AI Makes
- Premature abstraction: creating factory patterns, strategy classes, and generic registries for one-time operations.
- Violating the Single Responsibility Principle: writing "God classes" or 1,000-line utility files.
- Circular dependencies between modules.
- Leaking infrastructure details (e.g., raw database handles, AWS SDK clients) directly into domain entities.

### 15.2 Why Those Mistakes Are Problematic
- Premature abstractions add immense cognitive load and make subsequent modifications five times more difficult.
- God objects are impossible to test in isolation and become breeding grounds for merge conflicts.
- Circular dependencies cause runtime initialization crashes and memory leaks.

### 15.3 How They Should Be Prevented
- Follow the Rule of Three: write concrete code first; extract an abstraction only after three distinct implementations duplicate logic.
- Enforce strict dependency rules: domain models must not import infrastructure or HTTP transport layers.
- Limit function and module sizes (e.g., maximum 50 lines per function, 300 lines per module).

### 15.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST NOT** introduce abstractions before the third concrete use case, **MUST** enforce unidirectional dependency flow from transport to domain, and **MUST** keep modules focused on a single responsibility.

### 15.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Enterprise over-engineering for a trivial string transform
interface IStringTransformerStrategy<TInput, TOutput> {
  transform(input: TInput): TOutput;
}
class StringTransformerFactory {
  static createCapitalizer(): IStringTransformerStrategy<string, string> {
    return new class implements IStringTransformerStrategy<string, string> {
      transform(input: string): string {
        return input.toUpperCase();
      }
    };
  }
}
// Usage requires 4 layers of mental gymnastics
const result = StringTransformerFactory.createCapitalizer().transform('hello');

// ✅ CORRECT: Clean, direct, self-explanatory utility
export function capitalize(text: string): string {
  if (!text) return '';
  return text.charAt(0).toUpperCase() + text.slice(1);
}
```

---

## 16. Dependency Management

### 16.1 Common Mistakes AI Makes
- Adding third-party packages for trivial tasks easily done with standard language libraries (e.g., importing `lodash.debounce` when a 10-line native function suffices, or importing `is-odd`).
- Adding packages with unvetted maintenance health, abandoned GitHub repos, or known critical CVEs.
- Adding duplicate libraries (e.g., installing `axios` when `fetch` or `got` is already in `package.json`).
- Committing loose version constraints without maintaining lockfile integrity.

### 16.2 Why Those Mistakes Are Problematic
- Every dependency expands the attack surface for supply chain compromises (e.g., event-stream, left-pad, xz utils).
- Inflates bundle size, increasing build times and network payload sizes.
- Unmaintained libraries lead to dependency hell when updating major framework versions.

### 16.3 How They Should Be Prevented
- Before adding any package, check whether the project's existing dependencies or native language APIs can solve the problem.
- Verify package health: maintenance status, download velocity, license compatibility, and open CVEs.
- Always check `package.json` to reuse existing utilities.

### 16.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST NOT** install dependencies for operations natively supported by modern runtimes, **MUST** check existing project dependencies before introducing new ones, and **MUST** verify license and CVE status.

### 16.5 Bad vs. Correct Approach
```json
// ❌ BAD: Bloated package.json with redundant packages for native features
{
  "dependencies": {
    "left-pad": "^1.3.0",
    "is-even": "^1.0.0",
    "uuid": "^9.0.0",
    "axios": "^1.6.0" // Redundant when project already uses fetch!
  }
}
```
```typescript
// ✅ CORRECT: Zero extra dependencies using modern native APIs
// Native String.padStart replaces left-pad:
const formatted = "42".padStart(5, "0"); // "00042"

// Native modulo replaces is-even:
const isEven = (n: number): boolean => n % 2 === 0;

// Native Web Crypto replaces uuid:
const id = crypto.randomUUID();

// Native fetch replaces axios:
const response = await fetch('/api/data', {
  headers: { 'Content-Type': 'application/json' }
});
```

---

## 17. Testing

### 17.1 Common Mistakes AI Makes
- Writing tests that assert on internal implementation details (e.g., testing that a private method was called 3 times) rather than observable behavior.
- Writing tautological or "mock-heavy" tests where the test only tests the mock setup.
- Generating tests with non-deterministic dependencies (relying on live network calls, current system time, or random generators without fixed seeds).
- Skipping edge-case tests (testing only valid happy-path strings, skipping empty strings, negative numbers, boundary limits).

### 17.2 Why Those Mistakes Are Problematic
- Implementation-detail tests break during harmless refactoring, creating maintenance friction and false alarms.
- Tautological tests provide 100% false confidence without catching genuine regression bugs.
- Flaky, non-deterministic tests erode developer trust in CI/CD pipelines.

### 17.3 How They Should Be Prevented
- Test observable contracts: inputs and outputs at public module boundaries.
- Mock only external boundaries (HTTP networks, email senders, payment processors), never internal domain logic.
- Test boundary limits: null, zero, empty collection, maximum capacity, timeout failure.

### 17.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** test public observable behavior rather than internal private implementation, **MUST** test boundary edge cases and failure modes, and **MUST** guarantee test determinism.

### 17.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Tautological test testing internal mock wiring, zero real validation
it('processes user', async () => {
  const mockService = { doInternalCalculation: jest.fn().mockReturnValue(42) };
  const controller = new UserController(mockService);
  await controller.handle();
  // Only asserts that mock was called, doesn't verify if calculation was correct!
  expect(mockService.doInternalCalculation).toHaveBeenCalledTimes(1);
});

// ✅ CORRECT: Behavioral test with edge cases and deterministic assertion
describe('DiscountCalculator', () => {
  const calculator = new DiscountCalculator();

  it('applies 10% tier discount for orders exceeding $100', () => {
    const total = calculator.calculateTotal({ subtotal: 150, customerTier: 'GOLD' });
    expect(total).toBe(135.00);
  });

  it('handles edge case: zero or negative order subtotal gracefully', () => {
    expect(() => calculator.calculateTotal({ subtotal: 0, customerTier: 'REGULAR' }))
      .toThrow(InvalidOrderError);
    expect(() => calculator.calculateTotal({ subtotal: -10, customerTier: 'REGULAR' }))
      .toThrow(InvalidOrderError);
  });

  it('preserves boundary: exact $100 threshold does not trigger discount', () => {
    const total = calculator.calculateTotal({ subtotal: 100, customerTier: 'GOLD' });
    expect(total).toBe(100.00);
  });
});
```

---

## 18. DevOps and Deployment

### 18.1 Common Mistakes AI Makes
- Generating Dockerfiles that run containers as the `root` user.
- Neglecting multi-stage Docker builds, resulting in 1.5GB production images containing compilers, test runners, and source maps.
- Omitting container healthcheck and readiness endpoints (`/healthz`, `/readyz`).
- Hardcoding ports, database URLs, and file paths inside Docker configurations.
- Missing graceful shutdown signal handling (`SIGTERM`, `SIGINT`), causing in-flight request termination during rolling deployments.

### 18.2 Why Those Mistakes Are Problematic
- Running as root gives attackers full host root privileges if container breakout occurs.
- Huge images slow CI/CD deploy times from 2 minutes to 20 minutes and consume excess cloud storage.
- Missing graceful shutdown drops active user transactions and corrupts ongoing database writes during updates.

### 18.3 How They Should Be Prevented
- Create dedicated unprivileged system users (`USER node`, `USER appuser`) in Docker containers.
- Use multi-stage builds to export only the production runtime binary and node_modules.
- Implement HTTP `/healthz` probes and trap `SIGTERM` to close database connection pools gracefully.

### 18.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** run production containers as unprivileged users, **MUST** use multi-stage builds for minimal image size, and **MUST** implement graceful shutdown for `SIGTERM` signals.

### 18.5 Bad vs. Correct Approach
```dockerfile
# ❌ BAD: Runs as root, copies development tools into production, single stage
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]

# ✅ CORRECT: Multi-stage build, unprivileged user, minimal final footprint
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && npm prune --production

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
# Run as built-in non-root user
USER node
COPY --chown=node:node package*.json ./
COPY --chown=node:node --from=builder /app/node_modules ./node_modules
COPY --chown=node:node --from=builder /app/dist ./dist

EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:3000/healthz || exit 1
CMD ["node", "dist/server.js"]
```

---

## 19. Configuration and Environment Variables

### 19.1 Common Mistakes AI Makes
- Accessing `process.env.VARIABLE_NAME` directly throughout application business logic instead of a centralized, validated config module.
- Failing to validate environment variables at startup, allowing apps to crash hours later when a missing variable is finally accessed.
- Committing `.env` files containing live credentials to git repositories.
- Hardcoding fallback defaults for security-critical parameters (e.g., `const JWT_SECRET = process.env.JWT_SECRET || 'secret123'`).

### 19.2 Why Those Mistakes Are Problematic
- Insecure fallback defaults result in production systems running with publicly known secrets.
- Missing variables causing runtime crashes midway through customer transactions create severe incidents.
- Scattered `process.env` calls make it impossible to audit which environment variables the application requires.

### 19.3 How They Should Be Prevented
- Validate all environment variables at application startup using a strict schema (e.g., Zod).
- Fail immediately on boot (`process.exit(1)`) if any required environment variable is missing or malformed.
- Centralize all configurations in a single typed `config` object.
- Never provide default fallbacks for secrets or tokens.

### 19.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** validate all environment variables at startup via a centralized schema, **MUST NOT** provide insecure default fallbacks for secrets, and **MUST NOT** access `process.env` outside the config module.

### 19.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Scattered process.env, dangerous fallback secret, crashes late at runtime
function signAuthToken(payload: object) {
  // DISASTROUS: Falls back to a well-known weak secret!
  const secret = process.env.AUTH_SECRET || 'supersecret123';
  return jwt.sign(payload, secret);
}

// ✅ CORRECT: Centralized validation schema, fails immediately on startup
import { z } from 'zod';

const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  AUTH_SECRET: z.string().min(32, 'AUTH_SECRET must be at least 32 characters long'),
  REDIS_URL: z.string().url().optional()
});

function loadConfig() {
  const result = EnvSchema.safeParse(process.env);
  if (!result.success) {
    console.error('❌ FATAL: Invalid application environment configuration:');
    console.error(result.error.format());
    process.exit(1); // Fail-fast before accepting traffic
  }
  return result.data;
}

export const config = loadConfig();
```

---

## 20. Logging and Monitoring

### 20.1 Common Mistakes AI Makes
- Using raw `console.log()` statements instead of structured, leveled loggers.
- Logging sensitive PII and secrets (passwords, credit card PANs, auth tokens, SSNs).
- Missing correlation / request IDs, making it impossible to trace distributed multi-step transactions across services.
- Logging either too much (filling disks with debug logs in production) or too little (logging "An error occurred" with no context).

### 20.2 Why Those Mistakes Are Problematic
- Logging PII violates GDPR, HIPAA, and PCI-DSS, exposing organizations to massive regulatory fines.
- Without correlation IDs, debugging a failure across multiple microservices requires manual log timestamp correlation.
- Unstructured console logs cannot be ingested, indexed, or queried effectively by log management platforms (Datadog, CloudWatch, Loki).

### 20.3 How They Should Be Prevented
- Use structured JSON loggers (`pino`, `winston`, `structlog`).
- Implement automated log redaction filters for keys matching `/password|token|secret|creditcard|authorization/i`.
- Propagate `x-request-id` correlation headers across all logs and downstream HTTP calls.

### 20.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST** use structured JSON logging with appropriate severity levels, **MUST** automatically redact sensitive data and credentials, and **MUST** tag all request logs with correlation IDs.

### 20.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Unstructured logging, leaking user password and auth token
app.post('/login', async (req, res) => {
  console.log('User attempting login:', req.body); // Leaks password in plaintext!
  const user = await authService.login(req.body.email, req.body.password);
  console.log('User logged in with token: ' + user.token); // Leaks session token!
  res.json(user);
});

// ✅ CORRECT: Structured JSON logging, automated redaction, correlation tracking
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  redact: {
    paths: ['req.headers.authorization', 'password', 'token', 'creditCard'],
    censor: '[REDACTED]'
  },
  formatters: {
    level: (label) => ({ level: label })
  }
});

app.post('/login', async (req, res) => {
  const reqId = req.headers['x-request-id'] || crypto.randomUUID();
  const log = logger.child({ reqId, email: req.body.email });

  log.info('Login attempt initiated');
  const user = await authService.login(req.body.email, req.body.password);
  log.info({ userId: user.id }, 'Login successful');
  
  res.json({ success: true, data: user });
});
```

---

## 21. Documentation

### 21.1 Common Mistakes AI Makes
- Writing redundant, tautological comments that merely restate the code syntax (`// set counter to zero: counter = 0;`).
- Leaving comments that contradict the code because the code was updated while the comment was ignored.
- Failing to document Architecture Decision Records (ADRs) explaining *why* an unusual technical choice was made.
- Omitting instructions on how to set up, seed, or test the application.

### 21.2 Why Those Mistakes Are Problematic
- Tautological comments clutter files and distract developers from genuine complexity.
- Outdated, contradictory comments are worse than no comments, actively misleading engineers during emergencies.
- Missing "why" documentation leads future engineers to delete subtle bug workarounds or re-introduce past security bugs.

### 21.3 How They Should Be Prevented
- Write code that is self-documenting through clear naming and small functions.
- Reserve comments strictly for explaining *intent*, *business constraints*, *workarounds*, and *security considerations*.
- Maintain lightweight Architecture Decision Records (ADRs) for significant architectural forks.

### 21.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST NOT** generate tautological comments, **MUST** document the "why" and non-obvious business/security rationale, and **MUST** update existing documentation when modifying related code.

### 21.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Tautological comments, missing rationale for mysterious constant
// This function gets the user by id
function getUser(id: string) {
  // Return the user from database
  return db.users.find(id);
}

// Multiply by 1.15
const val = price * 1.15;

// ✅ CORRECT: Self-explanatory code with comments reserved for "why"
function getUserById(userId: string): Promise<User | null> {
  return db.users.findUnique({ where: { id: userId } });
}

/**
 * Applies statutory New York state and local sales tax surcharge.
 * Mandated by NY Dept of Taxation Bulletin TB-ST-740 (revised 2024).
 */
const NY_SALES_TAX_SURCHARGE_RATE = 0.15;
const finalPrice = basePrice * (1 + NY_SALES_TAX_SURCHARGE_RATE);
```

---

## 22. Refactoring and Maintenance

### 22.1 Common Mistakes AI Makes
- Combining refactoring with functional behavior changes in the same diff.
- Changing public function signatures, response shapes, or database columns without mapping all callers.
- Performing massive, unprompted reformatting or code rewrites that erase git blame and introduce silent bugs.
- Deleting "unused" parameters or functions that are actually called dynamically or by external consumers.

### 22.2 Why Those Mistakes Are Problematic
- Mixing refactoring with behavior changes makes it impossible for reviewers to isolate which line caused a production regression.
- Modifying signatures without updating all callers causes runtime `TypeError: undefined is not a function` in unexercised branches.
- Giant diffs overwhelm code review capacity, guaranteeing that subtle bugs slip into production.

### 22.3 How They Should Be Prevented
- Strictly separate pure refactoring commits (zero behavior change) from feature modification commits.
- Map every consumer across the repository using static analysis or grep before altering any interface.
- Keep diffs small, surgical, and bounded strictly to the requested task.

### 22.4 The Correct Rule an AI Should Follow
> **Rule**: An AI agent **MUST NOT** mix refactoring with behavioral changes, **MUST** verify and map all downstream consumers before modifying any interface, and **MUST** keep diffs surgical and minimal.

### 22.5 Bad vs. Correct Approach
```typescript
// ❌ BAD: Renames exported function and changes return shape without updating callers!
// Original: export function getCustomer(id: string): Promise<Customer>
// Modified by AI:
export function fetchClient(clientId: string): Promise<{ data: Customer; status: number }> {
  // Broke 14 callers across the codebase that expected getCustomer() returning Customer!
  ...
}

// ✅ CORRECT: Maintain backward compatibility or deprecate gracefully
/**
 * @deprecated Use `getCustomerProfile` instead. Will be removed in v3.0.
 */
export async function getCustomer(id: string): Promise<Customer> {
  const profile = await getCustomerProfile(id);
  return profile.customer;
}

export async function getCustomerProfile(id: string): Promise<CustomerProfileResponse> {
  return customerRepository.findById(id);
}
```
