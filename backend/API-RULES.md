# API Design & Contract Rules

> **Standard**: RESTful Conventions, HTTP Status Semantics, Idempotency, and Contract Envelopes.

---

## Standard HTTP Status Code Matrix

| Status Code | Semantic Meaning | Applicable Scenario |
| :--- | :--- | :--- |
| **200 OK** | Successful read or non-creation update | `GET`, `PUT`, `PATCH` responses returning data |
| **201 Created** | Resource successfully created | `POST` creating an entity. MUST include created object or Location header |
| **204 No Content** | Successful request, empty body | `DELETE` or state-change mutations returning no payload |
| **400 Bad Request** | Malformed syntax or invalid parameter | Schema failure, unparseable JSON |
| **401 Unauthorized**| Missing or invalid credentials | Authentication required or expired token |
| **403 Forbidden** | Authenticated user lacks permission | Authorization / RBAC failure (valid identity, insufficient privilege) |
| **404 Not Found** | Resource does not exist | Unknown URL or missing entity (also prevents IDOR leaks) |
| **409 Conflict** | State collision or duplicate key | Unique constraint violation (e.g. email already exists) |
| **422 Unprocessable**| Semantic validation failure | Valid JSON syntax but violates business validation rules |
| **429 Too Many Req** | Rate limit exceeded | Token bucket / rate limit exhausted |
| **500 Internal Error**| Unexpected server failure | Database crash, unhandled exception |

---

## Rule API-01: Never Return 200 OK for Business or Validation Errors [MUST]
- **Rule**: If a request fails due to missing parameters, schema validation, or authorization denials, the response MUST use a `4xx` or `5xx` status code. Never return HTTP 200 with `{ "error": "Invalid password" }`.
- **Reason**: HTTP status codes are standard metadata evaluated by API gateways, CDN caches, reverse proxies, and health monitors. Returning 200 for errors fools infrastructure into recording broken requests as 100% healthy.
- **Risk**: Blind monitoring: alerting systems fail to trigger during outages because all error responses return HTTP 200.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: HTTP 200 for an authorization failure
  res.status(200).json({ success: false, error: 'Unauthorized' });

  // ✅ CORRECT: Semantic 401 Unauthorized
  res.status(401).json({
    success: false,
    error: {
      code: 'AUTHENTICATION_REQUIRED',
      message: 'A valid Bearer token is required to access this resource'
    }
  });
  ```
- **Exceptions**: GraphQL endpoints, where the protocol specification specifies 200 OK with an `errors` array in the JSON body.

---

## Rule API-02: Consistent Response Envelopes [MUST]
- **Rule**: All REST endpoints across the system MUST return predictable, standardized response envelopes.
- **Reason**: Inconsistent response envelopes force frontend clients to write defensive boilerplate per endpoint (`res.data` vs `res.items` vs raw arrays).
- **Risk**: Client-side crashes caused by unexpected payload structures.
- **Correct Approach**:
  ```typescript
  // Single Entity Success:
  {
    "success": true,
    "data": { "id": "usr_123", "name": "Alice" }
  }

  // Paginated Collection Success:
  {
    "success": true,
    "data": [ ...items ],
    "meta": {
      "page": 1,
      "limit": 20,
      "total": 450,
      "totalPages": 23
    }
  }

  // Error Envelope (See ERROR-HANDLING-RULES.md):
  {
    "success": false,
    "error": {
      "code": "VALIDATION_FAILED",
      "message": "The payload contains invalid fields",
      "details": [{ "field": "email", "issue": "Invalid email format" }],
      "correlationId": "req_8f7b2c"
    }
  }
  ```
- **Exceptions**: Binary file downloads, image streams, and raw CSV exports.

---

## Rule API-03: Mandatory Pagination on All Collection Endpoints [MUST]
- **Rule**: Any endpoint returning an array of entities MUST support pagination (`page` and `limit`, or cursor-based `before` and `after`) and MUST enforce a hard maximum limit (e.g. maximum 100 items per page).
- **Reason**: Unbounded queries will eventually return tens of thousands of rows as the database grows, causing memory exhaustion and timeout crashes.
- **Risk**: Database out-of-memory crashes and severe latency degradation.
- **Correct Approach**:
  ```typescript
  app.get('/api/v1/orders', async (req, res) => {
    const page = Math.max(1, parseInt(req.query.page as string, 10) || 1);
    const requestedLimit = parseInt(req.query.limit as string, 10) || 20;
    const limit = Math.min(100, Math.max(1, requestedLimit)); // Strict 100 cap

    const { items, total } = await orderService.listOrders({ page, limit });
    res.json({ success: true, data: items, meta: { page, limit, total, totalPages: Math.ceil(total / limit) } });
  });
  ```
- **Exceptions**: Static, small lookup tables guaranteed to remain under 50 records (e.g. country codes).

---

## Rule API-04: Idempotency Keys on State-Mutating Operations [CONTEXT-DEPENDENT]
- **Rule**: Mission-critical mutation endpoints (payments, order placement, funds transfers) MUST accept an `Idempotency-Key` header and cache previous results in Redis or database to prevent duplicate execution upon retry.
- **Applicability**: Mandatory for financial transactions and order placement; optional for non-financial operations.
- **Reason**: Network timeouts often prompt client retry logic when the server has already processed the request.
- **Risk**: Customer double-charging and duplicate order dispatch.
- **Correct Approach**:
  ```typescript
  app.post('/api/v1/charges', async (req, res) => {
    const idempotencyKey = req.headers['idempotency-key'] as string;
    if (!idempotencyKey) {
      return res.status(400).json({ error: 'Idempotency-Key header is required' });
    }

    const cached = await redis.get(`idempotency:${idempotencyKey}`);
    if (cached) return res.status(200).json(JSON.parse(cached));

    const result = await paymentService.charge(req.body);
    await redis.set(`idempotency:${idempotencyKey}`, JSON.stringify(result), 'EX', 86400);
    return res.status(201).json(result);
  });
  ```
- **Exceptions**: Naturally idempotent `PUT` (full replacement) or `DELETE` operations.
