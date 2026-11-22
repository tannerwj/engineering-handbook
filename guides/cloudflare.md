# Operate a Cloudflare application

Use the relevant section when changing a Worker, D1 query, binding, migration, or release. Check platform behavior against the docs and the project’s configured compatibility date.

## Platform (Cloudflare Workers)

**Response handling.** Never `res.clone()` after the body has been transferred.
It 500s downstream handlers. Rebuild via
`new Response(res.body, { status, statusText, headers })`.

**Accounts.** Pin the account ID in the Worker config, and bind CLI auth to the
project directory when you use more than one Cloudflare account. A command
should fail rather than land in the wrong account.

**Migrations.**
- Additive and backward-compatible: the previous code version must run on the
  new schema, so a code rollback never needs a schema rollback.
- Snapshot first (D1 Time Travel bookmark).
- `wrangler deploy` doesn't apply them. Make the deploy script apply
  migrations (by binding name) before deploying, so nobody forgets.

**Deploys.**
- Validate first: dry-run build plus binding check.
- Know the undo before you deploy: the current version ID and the exact
  rollback command.
- After deploying, verify routes, crons, secrets and bindings are what you
  expect (especially after changing deploy tools), and curl new/changed
  endpoints live, including logged-out/gated responses. Routes can be
  unmounted even when tests pass.
- A brand-new hostname or route can 404/500 for a few seconds. Retry before
  declaring the deploy broken.
- Verify the served bundle hash matches the build (cache-bust the first fetch);
  edge HTML can lag by a minute.

## Observability (Cloudflare)

**Enable it in config.** Logs 100%, traces sampled, Issues on, source maps
uploaded (Wrangler 4.134+ for Issues). Tune sampling to traffic.

```jsonc
{
  "upload_source_maps": true,
  "observability": {
    "enabled": true,
    "logs": { "enabled": true, "invocation_logs": true, "head_sampling_rate": 1 },
    "traces": { "enabled": true, "head_sampling_rate": 0.1 },
    "issues": { "enabled": true }
  }
}
```

**Rules.**
- Log structured objects with stable event names plus service, environment,
  release, request/job ID, outcome and duration. Aggregate by route template.
- Log the completion of every request/job attempt; distinguish "enqueued" from
  "finished".
- Add custom spans (`tracing.enterSpan` from `cloudflare:workers`) for
  meaningful stages. CPU-only spans show 0 ms; profile computation instead.
- Invocation success isn't HTTP or business success, and wall time includes
  `waitUntil` (it isn't browser latency). Measure the question you're asking.
- Give agents a read path: the telemetry query API or Observability MCP, with
  saved queries for failures, slow routes and pre/post-release comparison.
  Native calculations already weight sampling; don't scale counts again.
- Prove it after each deploy: fetch a live success and a controlled failure
  through that read path. Missing data is unknown, not healthy.
- Alerts need an owner, a runbook and tested delivery.

Docs: [configuration](https://developers.cloudflare.com/workers/wrangler/configuration/),
[custom spans](https://developers.cloudflare.com/workers/observability/traces/custom-spans/),
[metrics](https://developers.cloudflare.com/workers/observability/metrics-and-analytics/),
[telemetry query API](https://developers.cloudflare.com/api/resources/workers/subresources/observability/subresources/telemetry/methods/query/),
[Local Explorer](https://developers.cloudflare.com/workers/local-development/local-explorer/).

## Apps other people deploy

**One click.** Ship a template config that deploys with no edits: derive what
you can at runtime (e.g. base URL from the serving host) and generate what
must exist (keys) on first use, with an explicit override. List required
secrets in `.dev.vars.example`. The deploy script applies migrations.

**First run.** See the bootstrap rules in [Security](security.md).

**A way back.** Deploy-button installs are copies, not GitHub forks. Ship:
- in-app feedback links (issue forms, pre-filled with the version);
- a version-based update notice that can be switched off;
- "staying up to date" docs (add `upstream`, merge, push).

## D1 and Worker cost

For a read that grows with tenant history, name the unit of growth. Run the operation on small and large synthetic tenants under comparable conditions. Capture query count, rows read, Worker CPU and elapsed latency as relevant. Choose an index or batch size from the query shape and actual binding limits. Exercise the last input that fits one batch and the first that needs two. Verify equivalent records and tenant scope before comparing cost. [D1 metrics](https://developers.cloudflare.com/d1/observability/metrics-analytics/)

For cached responses, list every input that changes the representation. Request HTML and Markdown from the same public URL, change the content, and deploy a changed template against unchanged content. Verify the expected body and cache behavior in each case.

## Tools under evaluation

**Effect.** A provider read with distinct authentication, throttling and deadline outcomes is a candidate for typed errors and bounded retries. Compare plain TypeScript and Effect on the same response sequences. Measure calls, final outcome, Worker bundle size and the effort to change retry policy correctly. An in-process retry needs a running invocation. Durable recovery needs persisted state. [Effect scheduling](https://effect.website/docs/v4/scheduling/using-schedules)

**Alchemy.** For a disposable Worker and D1 stage, examine the plan, provision the stage, verify a second plan converges and inspect resources retained after teardown. Identify account, stage, state backend and resource ownership before starting. With Cloudflare-hosted state, the first plan can bootstrap shared resources. Record the result before adopting Alchemy for an existing deployment. [Alchemy state store](https://alchemy.run/state-store/)
