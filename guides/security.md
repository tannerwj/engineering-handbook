# Protect identities and data

For each operation, identify the actor, resource owner, credential and side effect. Try a denied operation on synthetic records and verify both the response and the stored state.

## Select the trust boundary

Identify the component that owns the data and can enforce the decision. Include background jobs, imports and administrative paths when they can perform the same operation. For a local utility handling sensitive exports, check which files it creates, who can read them and whether errors or diagnostics reveal their contents.

Keep credentials outside source control. Use the environment’s secret store, document required credential names, and check output, errors and fixtures for accidental disclosure.

## When a service accepts credentials or network input

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

For browser cookies and response headers, see [hosted web services](../profiles/web-services.md).
