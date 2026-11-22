# Protect identities and data

For each operation, identify the actor, resource owner, credential and side effect. Try a denied operation on synthetic records and verify both the response and the stored state.

## Security

**Response headers.** Every public worker/site sends on all responses:
- `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: strict-origin-when-cross-origin`
- Clickjacking defense: `X-Frame-Options: DENY` or CSP `frame-ancestors 'none'`
- `Permissions-Policy` denying what the app doesn't use, and
  `Cross-Origin-Opener-Policy: same-origin`
- A real `Content-Security-Policy` when serving HTML with scripts: no inline
  script or style. Wire behavior through `data-*` attributes and a bundled
  script; frameworks that inject inline scripts need nonces, not
  `'unsafe-inline'`.
- `Cache-Control: no-store` on authenticated HTML.

Set headers in one place: Hono middleware (`app.use("*", ...)`) or a
`withSecurityHeaders(res)` wrapper on the exported `fetch`.

**Sessions.** Cookies use the `__Host-` prefix, `HttpOnly`, `Secure`,
`SameSite=Lax`. SameSite treats sibling subdomains as same-site, so every
cookie-authenticated state change also checks `Sec-Fetch-Site` (fallback:
`Origin`) against the app's own origin.

**Tokens and secrets in transit.**
- One-time tokens (invites, resets, codes): 256-bit random, hashed at rest,
  short TTL, consumed in the same statement that checks them
  (`UPDATE … WHERE used = 0 RETURNING …`) so two requests can't both win.
- Compare secrets in constant time.
- Bearer tokens are audience-bound: an API accepts only tokens issued for it,
  never a token minted for some other app.

**Abuse.** Rate-limit credential and auth endpoints per client IP. Hash IPs
before logging them.

**Server-side fetches of user-supplied URLs** (webhooks, metadata documents,
avatars): https only, size and time caps, no private/loopback addresses,
and treat the response as untrusted input.

**First-run bootstrap.** Creating the first admin needs a secret set at deploy
time, only works while no users exist, and inserts conditionally ("only if
still empty") so a race can't create two. After that the route is gone.

**Secrets.** Never commit credentials; `abacus secrets` (gitleaks) gates it.
Never print secret values (check length only). Upload them via stdin or a
temp secrets file that's deleted right after. Document required secrets, without
values, in `.dev.vars.example`.
