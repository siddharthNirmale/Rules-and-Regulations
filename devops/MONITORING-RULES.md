# Observability, Logging & Monitoring Rules

> **Scope**: Telemetry, structured JSON logging, distributed tracing, APM metrics, and alerting hygiene.

---

## The Observability Pillars (M-E-L-T)
Production systems MUST provide telemetry across four core pillars:
1. **Metrics**: Aggregable numerical timeseries (e.g. CPU, RAM, HTTP request counts, latency histograms).
2. **Events**: Discrete point-in-time domain occurrences (e.g. user registered, deployment finished).
3. **Logs**: Structured JSON records of operational details and exceptions.
4. **Traces**: Distributed request propagation across microservices and database calls.

---

## Rule MON-01: Exclusively Emit Structured JSON Logs [MUST]
- **Rule**: Production services MUST emit structured JSON logs to standard output (`stdout`) using leveled loggers (`pino`, `winston`, `structlog`). Unstructured plain text strings via `console.log()` are prohibited.
- **Reason**: Log aggregation systems (Datadog, AWS CloudWatch, ElasticSearch, Loki) can index, filter, and alert on JSON fields automatically. Unstructured string logs require expensive regex parsing and cannot be queried reliably.
- **Risk**: Inability to query logs during production outages.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Raw console log string
  console.log(`User ${userId} paid ${amount} on order ${orderId}`);

  // ✅ CORRECT: Structured JSON log with typed context fields
  logger.info({
    event: 'ORDER_PAYMENT_PROCESSED',
    userId,
    orderId,
    amountCents: amount * 100,
    currency: 'USD'
  }, 'Order payment processed successfully');
  ```
- **Exceptions**: Local interactive terminal scripts or CLI prompt outputs.

---

## Rule MON-02: Mandatory Automated Redaction of Sensitive Data [MUST]
- **Rule**: Logger configurations MUST define automated redaction masks that sanitize keys matching `password`, `token`, `secret`, `authorization`, `creditCard`, `cvv`, `ssn`, and session cookies.
- **Reason**: Accidental logging of authentication tokens or user credit card numbers violates PCI-DSS and GDPR regulations, exposing the organization to legal penalties.
- **Risk**: Permanent credential leaks in log aggregation databases accessible to broad engineering teams.
- **Correct Approach**:
  ```typescript
  // ✅ CORRECT: Pino automated redaction filter
  import pino from 'pino';

  export const logger = pino({
    level: process.env.LOG_LEVEL || 'info',
    redact: {
      paths: [
        'password',
        '*.password',
        'req.headers.authorization',
        'req.headers.cookie',
        'token',
        'creditCard',
        'cvv'
      ],
      censor: '[REDACTED]'
    }
  });
  ```
- **Exceptions**: None.

---

## Rule MON-03: Propagate Correlation IDs Across All Logs and RPC Calls [MUST]
- **Rule**: Every incoming HTTP request MUST be assigned a unique Correlation / Request ID (via `x-request-id` header or `crypto.randomUUID()`). This correlation ID MUST be attached to every log statement emitted during the request and forwarded in the headers of all downstream HTTP or RPC calls.
- **Reason**: In distributed architectures, a single user click may trigger calls across five separate microservices. Without a correlation ID, tracing a failure requires manually guessing timestamps.
- **Risk**: Hours wasted attempting to isolate which log lines belong to which failing user session.
- **Correct Approach**:
  ```typescript
  app.use((req, res, next) => {
    const correlationId = (req.headers['x-request-id'] as string) || crypto.randomUUID();
    res.setHeader('x-request-id', correlationId);
    
    // Attach child logger containing correlationId to request object
    req.log = logger.child({ correlationId, path: req.path, method: req.method });
    next();
  });
  ```
- **Exceptions**: Offline batch worker scripts that process data without an external HTTP trigger (which should generate a `batchRunId`).

---

## Rule MON-04: Instrument the RED Method for Service Metrics [MUST]
- **Rule**: Backend services MUST track and expose metrics following the **RED Method**:
  1. **Rate**: Number of incoming requests per second.
  2. **Errors**: Number of failed requests per second (broken down by HTTP 4xx vs 5xx status codes).
  3. **Duration**: Distribution of request latencies (measuring p50, p90, p95, and p99 percentiles).
- **Reason**: The RED method provides immediate insight into system health and user experience. A spike in Duration or Errors indicates an active production incident.
- **Risk**: Silent degradation where a service slows to a 10-second response time without throwing explicit crashes.
- **Correct Approach**: Instrument routes with Prometheus or OpenTelemetry middleware (`prom-client`, `@opentelemetry/api`).
- **Exceptions**: Short-lived CLI commands.

---

## Rule MON-05: Actionable Alerting Hygiene (Zero Alert Fatigue) [SHOULD]
- **Rule**: Alerts SHOULD trigger on user-facing symptoms (e.g. 5xx error rate > 1% over 5 minutes, or p95 latency > 2000ms), NOT on noisy ephemeral internal conditions (e.g. a single CPU spike to 85% for 10 seconds). Every alert MUST link directly to a runbook describing the remediation steps.
- **Reason**: Alert fatigue occurs when engineers receive dozens of non-actionable notifications a day. Teams learn to ignore the alerts, guaranteeing that real outages are missed.
- **Risk**: Missed production incidents due to notification desensitization.
- **Exceptions**: Critical security events (e.g., secret exfiltration attempts or repeated admin brute-force lockouts), which MUST alert on first occurrence.
