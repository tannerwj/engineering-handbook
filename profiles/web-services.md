# Test a hosted web service

Use these practices for a browser-facing service with deployed environments. Projects using tester-army/e2e use the TypeScript browser adapter.

**User journeys.** tester-army/e2e (`e2e` + `@e2e-dev/web`):
deterministic tests only. Poll/wait for UI state before asserting, and let a
navigation settle before the next step (real network latency exposes races
that local runs hide). Test from the functionality/usability side: what the
user does, what they see, what breaks. Cover happy paths *and* error/empty
states; fix real bugs, don't adjust tests to pass. Scope locators to regions.

**Staging on the real platform.** Local dev misses rate limiters, real
latency, propagation and platform quirks. Run the E2E suite against a deployed
staging environment with its own database and its own resource IDs. Any
"reset data" step refuses to run against anything but staging.

**Gates are fast or they get skipped.** Keep the pre-deploy gates to a few
minutes and measure where the time goes before cutting anything. Run
independent checks side by side (each typecheck project, lint, unit tests,
scans). Run E2E files in parallel: every file owns the tenants and fixtures it
writes, and the few files that change install-wide state or compare a shared
log run one at a time on their own app instance alongside. Test-side data
setup goes through the app's own database binding, never a second process on
the same file. Built-in waits (deadlines, pacing, expiries) get a knob that
only the test environment sets and that can only lower the production value.

**Don't rerun what already passed.** Record each gate's pass against the exact
source snapshot (the git tree hash of the working tree, untracked files
included). A deploy skips gates already green for that tree, and a docs-only
difference doesn't rerun E2E (anything code imports, like prompt markdown, is
not docs). While working, run the E2E files that cover the change; the full
suite runs once, before merge or deploy.

**Prod smoke is a smoke test.** After a deploy, a run of a minute or two
proves the release is alive and wired: health, auth gating, security headers,
a real sign-in path, every API view answering with its shape, one write round
trip on a synthetic tenant. Exhaustive browser sweeps (every page and
viewport, link crawls, accessibility on every sheet) run locally on every gate
and against prod on demand.


## Browser security

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

Apply headers at the common response boundary and verify successful and error responses.

**Sessions.** Cookies use the `__Host-` prefix, `HttpOnly`, `Secure`,
`SameSite=Lax`. SameSite treats sibling subdomains as same-site, so every
cookie-authenticated state change also checks `Sec-Fetch-Site` (fallback:
`Origin`) against the app's own origin.

