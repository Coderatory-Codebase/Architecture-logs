## 16. Security Architecture --- System-Wide

Security isn't one module --- it's layered across every request, on both
sides.

### Backend layers

  -----------------------------------------------------------------------
  Layer                               Measure
  ----------------------------------- -----------------------------------
  **Transport**                       HTTPS enforced in production;
                                      cookie `secure` flag already
                                      conditional on `ENV` (existing
                                      pattern)

  **Response headers**                `helmet()` already in place ---
                                      CSP, HSTS, X-Frame-Options,
                                      X-Content-Type-Options (existing)

  **Authentication**                  JWT in httpOnly cookies (existing)
                                      --- short-lived access token +
                                      longer refresh token, never exposed
                                      to JS (mitigates XSS token theft by
                                      design)

  **Authorization**                   `authorize.ts` enforced on every
                                      `hr/*` route, not just hidden in
                                      the frontend UI --- the server is
                                      the actual boundary

  **Input validation**                Joi schema on every mutating
                                      endpoint (existing pattern,
                                      extended to every new HR module)

  **Injection protection**            Mongoose uses parameterized queries
                                      by default (no raw string
                                      concatenation); Postgres/Neon
                                      queries use `$1, $2` placeholders
                                      exclusively --- string-built SQL is
                                      never used anywhere in the codebase

  **ID validation**                   New: `validateObjectId.ts`
                                      middleware rejects a malformed
                                      `:id` param with a clean `400`
                                      *before* it reaches Mongoose ---
                                      this closes a real gap found
                                      earlier in the Calendar module,
                                      where an invalid ID currently falls
                                      through to a generic `500` via an
                                      unhandled `CastError`

  **Rate limiting**                   Existing rate limiter extended to
                                      every write-heavy or
                                      brute-forceable endpoint: login,
                                      leave-apply, attendance check-in,
                                      payroll access

  **Secrets management**              `.env` excluded via `.gitignore`
                                      (already confirmed), separate
                                      `.env` per environment
                                      (dev/staging/prod); production
                                      secrets belong in the hosting
                                      platform's secret store
                                      (e.g. Render/Railway environment
                                      variables), never in a committed
                                      file

  **Sensitive-field encryption**      `Payslip.grossSalary` / `netSalary`
                                      encrypted at rest (AES-256 in the
                                      service layer before `save()`,
                                      decrypted only when returned to an
                                      authorized requester)

  **File uploads (Documents module)** MIME-type and size validated
                                      server-side before storage; files
                                      stored outside the web root;
                                      downloads served via short-lived
                                      signed URLs, never a permanent
                                      public link

  **CORS**                            Origin allowlist only (existing
                                      pattern) --- never `origin: '*'`

  **Audit trail**                     `AuditLog` entity (added in
                                      §12/§14) --- every write to
                                      Employee, Leave status, or Payroll
                                      records who did it and when

  **Dependency security**             `npm audit` run in CI on every PR
                                      (ties into §"Deployment
                                      Architecture" below)
  -----------------------------------------------------------------------

### Frontend layers

  -----------------------------------------------------------------------
  Layer                               Measure
  ----------------------------------- -----------------------------------
  **Token storage**                   No tokens in
                                      `localStorage`/`sessionStorage` ---
                                      the existing httpOnly-cookie
                                      pattern already prevents JS (and
                                      therefore XSS) from ever reading
                                      the token; this is intentionally
                                      *not* changed

  **XSS**                             React escapes rendered content by
                                      default (existing);
                                      `dangerouslySetInnerHTML` is never
                                      used for user-generated content
                                      such as announcement bodies

  **CSRF**                            `sameSite: strict` cookies
                                      (existing) already block the common
                                      cross-site cases; a CSRF token is
                                      added only if a legitimate
                                      cross-site flow is ever introduced

  **Environment variables**           Only `NEXT_PUBLIC_*` values
                                      (e.g. API base URL) are exposed to
                                      the browser bundle --- no secret
                                      ever gets a `NEXT_PUBLIC_` prefix

  **Dependency security**             `npm audit` / Dependabot alerts
                                      enabled on the frontend repo too
  -----------------------------------------------------------------------
