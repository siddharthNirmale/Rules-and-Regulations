# Backend Error Handling & Exception Management Rules

> **Scope**: Exception hierarchies, operational vs programmer errors, centralized error handling, and safe client sanitization.

---

## Error Classification
Backend systems MUST distinguish between two fundamentally distinct classes of errors:

1. **Operational / Validation Errors**: Expected failure modes during normal runtime (e.g. invalid form input, user not found, expired session, credit card declined). These MUST be caught, handled cleanly, and translated into semantically accurate HTTP `4xx` responses.
2. **Programmer / System Errors (Bugs)**: Unexpected defects in the software (e.g. `TypeError`, division by zero, database driver connection timeout, assertion failure). These MUST be logged with complete context, mapped to a sanitized `500 Internal Server Error`, and trigger engineer alerting.

---

## Rule ERR-01: Zero Silent Error Swallowing [MUST]
- **Rule**: Catch blocks MUST NOT be empty or purely contain no-op comments. Every caught exception MUST be logged, handled with a fallback, or rethrown.
- **Reason**: Swallowing exceptions hides failing subsystems, causes ghost data corruption, and makes production issues impossible to troubleshoot.
- **Risk**: Unreported data loss and cascading silent failures.
- **Correct Approach**:
  ```typescript
  // ❌ S1 REJECTION: Silent swallowed exception
  try {
    await sendConfirmationEmail(user);
  } catch (err) {}

  // ✅ CORRECT: Logged with fallback retry queue
  try {
    await sendConfirmationEmail(user);
  } catch (err) {
    logger.warn({ err, userId: user.id }, 'Confirmation email failed to dispatch; enqueuing retry');
    await retryQueue.add('sendEmail', { userId: user.id });
  }
  ```
- **Exceptions**: Documented fire-and-forget operations that are explicitly safe to ignore (e.g. non-critical analytics ping).

---

## Rule ERR-02: Sanitize System Errors; Provide Actionable Validation Details [MUST]
- **Rule**: Error responses returned to clients MUST differentiate between system errors and validation errors:
  1. **System Errors (`500`)**: MUST NEVER contain stack traces, file paths, database query fragments, or library versions. Return a generic message with a `correlationId`. Full details MUST be logged internally only.
  2. **Validation Errors (`400`/`422`)**: MUST provide field-level issue descriptions (`error.details: [{ field, issue }]`) so the client UI can display actionable recovery hints to the user.
  3. **Authentication Failures (`401`)**: MUST return a generic message (`"Invalid email or password"`) to prevent username enumeration attacks.
- **Reason**: Leaking internal stack traces reveals vulnerabilities to attackers, while withholding validation details frustrates legitimate users who cannot determine which field was rejected.
- **Risk**: Information disclosure on 500s; user abandonment on vague 400s; account enumeration on 401s.
- **Correct Approach**:
  ```typescript
  app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
    const correlationId = (req.headers['x-request-id'] as string) || crypto.randomUUID();

    // Internal log retains full diagnostic details
    logger.error({ err, correlationId, path: req.path }, 'Request execution failed');

    // Case 1: Validation / Operational Error (Return actionable details)
    if (err instanceof ValidationError) {
      return res.status(400).json({
        success: false,
        error: {
          code: 'VALIDATION_FAILED',
          message: 'The submitted payload contains invalid fields',
          details: err.fieldErrors, // Actionable for UI forms!
          correlationId
        }
      });
    }

    // Case 2: Known Domain Error
    if (err instanceof DomainError) {
      return res.status(err.statusCode).json({
        success: false,
        error: { code: err.code, message: err.message, correlationId }
      });
    }

    // Case 3: Unexpected System Bug (Sanitized generic fallback)
    return res.status(500).json({
      success: false,
      error: {
        code: 'INTERNAL_SERVER_ERROR',
        message: 'An unexpected error occurred. Please contact support.',
        correlationId
      }
    });
  });
  ```
- **Exceptions**: Local development environments where `NODE_ENV === 'development'`.

---

## Rule ERR-03: Create a Formal Domain Error Hierarchy [MUST]
- **Rule**: Applications MUST establish an application-specific error hierarchy inheriting from standard `Error`, with typed properties for HTTP status code, machine-readable error code, and optional metadata.
- **Reason**: Using string-based error checking (`if (err.message.includes('not found'))`) is brittle, breaks on copy changes, and violates type safety.
- **Risk**: Fragile error handling and incorrect status code assignments.
- **Correct Approach**:
  ```typescript
  export abstract class ApplicationError extends Error {
    abstract readonly statusCode: number;
    abstract readonly errorCode: string;

    constructor(message: string) {
      super(message);
      Object.setPrototypeOf(this, new.target.prototype);
    }
  }

  export class NotFoundError extends ApplicationError {
    readonly statusCode = 404;
    readonly errorCode = 'RESOURCE_NOT_FOUND';
  }

  export class ConflictError extends ApplicationError {
    readonly statusCode = 409;
    readonly errorCode = 'RESOURCE_CONFLICT';
  }
  ```
- **Exceptions**: Tiny one-off utility scripts under 100 lines.

---

## Rule ERR-04: Attach Global Process Crash Handlers [MUST]
- **Rule**: Backend services MUST register top-level listeners for `unhandledRejection` and `uncaughtException` to log fatal crashes and trigger graceful process shutdown.
- **Reason**: An unhandled promise rejection in modern runtimes leaves the process in an undefined, corrupted state.
- **Risk**: Abrupt process crashes without log traces, leaving open database sockets and dropping in-flight user requests.
- **Correct Approach**:
  ```typescript
  process.on('uncaughtException', (err: Error) => {
    logger.fatal({ err }, 'CRITICAL: Uncaught exception encountered. Initiating emergency shutdown.');
    gracefulShutdown(1);
  });

  process.on('unhandledRejection', (reason: unknown) => {
    logger.fatal({ reason }, 'CRITICAL: Unhandled promise rejection encountered.');
    gracefulShutdown(1);
  });
  ```
- **Exceptions**: None.
