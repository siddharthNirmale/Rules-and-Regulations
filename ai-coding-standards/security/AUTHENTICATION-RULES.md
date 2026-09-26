# Authentication & Session Security Rules

> **Scope**: Password hashing, session lifecycle, JWT verification, multi-factor authentication (MFA), and brute-force defenses.

---

## Rule AUTH-01: Modern Adaptive Password Hashing Only [CONTEXT-DEPENDENT]
- **Rule**: Passwords MUST be hashed using **Argon2id** (OWASP recommended parameters: memory 64MB, iterations 3, parallelism 4) or **bcrypt** (work factor cost >= 12). Fast cryptographic hash functions (MD5, SHA-1, SHA-256, SHA-512) MUST NEVER be used for passwords.
- **Applicability**: Mandatory whenever the application stores user credentials.
- **Reason**: General-purpose hash functions are designed to be fast and cheap. Consumer GPUs compute billions of SHA-256 hashes per second, making offline dictionary attacks trivial. Adaptive password hashers enforce high memory and CPU cost to defeat GPU cracking rigs.
- **Risk**: Immediate compromise of all user accounts following any database dump leak.
- **Correct Approach**:
  ```typescript
  import argon2 from 'argon2';

  export async function hashPassword(password: string): Promise<string> {
    return argon2.hash(password, {
      type: argon2.argon2id,
      memoryCost: 65536, // 64 MB
      timeCost: 3,
      parallelism: 4
    });
  }

  export async function verifyPassword(hash: string, plainText: string): Promise<boolean> {
    return argon2.verify(hash, plainText);
  }
  ```
- **Exceptions**: Legacy systems during active credential migration (which MUST re-hash with Argon2id on next successful user login).

---

## Rule AUTH-02: Issue Session Tokens in Secure, HttpOnly Cookies [CONTEXT-DEPENDENT]
- **Rule**: Authentication and session tokens MUST be transmitted to browsers inside cookies configured with `HttpOnly`, `Secure`, and `SameSite=Lax` (or `Strict`), using the `__Host-` prefix in production. Tokens MUST NOT be stored in `localStorage` or `sessionStorage`.
- **Applicability**: Mandatory for browser-based web applications; native mobile apps should store tokens in secure OS Keychains.
- **Reason**: The `HttpOnly` flag prevents client-side JavaScript (`document.cookie`) from reading the cookie, mitigating token exfiltration via Cross-Site Scripting (XSS).
- **Risk**: Full account takeover via any minor XSS vulnerability in any third-party script.
- **Correct Approach**:
  ```typescript
  res.cookie('__Host-session', sessionToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    path: '/',
    maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
  });
  ```
- **Exceptions**: Native mobile applications (iOS/Android) communicating via Authorization header and storing tokens in the platform Keychain / Keystore.

---

## Rule AUTH-03: Regenerate Session IDs on Privilege Elevation (Fixation Defense) [CONTEXT-DEPENDENT]
- **Rule**: When a user transitions from an unauthenticated state to an authenticated state (or elevates privileges to admin), the application MUST regenerate the session ID and invalidate the old session identifier.
- **Applicability**: Mandatory for stateful session-based web applications.
- **Reason**: Session fixation occurs when an attacker tricks a victim into authenticating with a pre-set session ID known to the attacker.
- **Risk**: Unauthorized account access without knowing the user's password.
- **Correct Approach**:
  ```typescript
  req.session.regenerate((err) => {
    if (err) return next(err);
    req.session.userId = user.id;
    req.session.save((err) => {
      res.json({ success: true });
    });
  });
  ```
- **Exceptions**: Stateless token models where no session identifier exists on the server.

---

## Rule AUTH-04: Strict JWT Verification & Algorithm Pinning [CONTEXT-DEPENDENT]
- **Rule**: When validating JSON Web Tokens (JWTs), the verification library MUST:
  1. Explicitly specify the permitted signing algorithm (e.g. `['RS256']` or `['HS256']`).
  2. Reject the `none` algorithm unconditionally.
  3. Validate both `exp` (expiration) and `nbf` (not before) claims.
  4. Ensure access tokens have a short lifespan (maximum 15 minutes).
- **Applicability**: Mandatory whenever JSON Web Tokens are utilized.
- **Reason**: Attackers can forge tokens by altering the header to `alg: "none"` or tricking systems using RSA public keys as HMAC shared secrets.
- **Risk**: Complete authentication bypass without credentials.
- **Correct Approach**:
  ```typescript
  import jwt from 'jsonwebtoken';

  export function verifyAccessToken(token: string, publicKey: string) {
    return jwt.verify(token, publicKey, {
      algorithms: ['RS256'], // Pinned algorithm; rejects 'none' or 'HS256'
      maxAge: '15m'
    });
  }
  ```
- **Exceptions**: None when JWTs are used.

---

## Rule AUTH-05: Rate Limiting & Brute-Force Throttling on Auth Endpoints [MUST]
- **Rule**: Authentication endpoints (`/login`, `/register`, `/forgot-password`, `/verify-otp`) MUST be protected by IP-based and username-based rate limiters (e.g. maximum 5 failed attempts per 15-minute window before exponential delay or temporary lockout).
- **Reason**: Without rate limits, automated bots can test millions of credential combinations using distributed botnets.
- **Risk**: Credential stuffing and account takeover via brute-force enumeration.
- **Correct Approach**:
  ```typescript
  import rateLimit from 'express-rate-limit';

  export const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5,                   // Limit each IP to 5 requests per window
    standardHeaders: true,
    legacyHeaders: false,
    message: {
      success: false,
      error: { code: 'TOO_MANY_ATTEMPTS', message: 'Too many login attempts. Please try again later.' }
    }
  });
  ```
- **Exceptions**: Automated end-to-end test environments running in isolated sandboxes.
