# Operate a Cloudflare application

Use the relevant section when changing a Worker, D1 query, binding, migration, or release. Check platform behavior against the docs and the project’s configured compatibility date.

For account selection, local development, release commands and resource operations, start with the [project-scoped cf CLI guide](../profiles/cf-cli.md).

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

### Profile a deployed Worker

For workload design, before-and-after comparisons and deciding which optimization to keep, see [Improve performance from a profile](profiling.md).

Use on-demand profiling when a real workload shows elevated Worker CPU time or memory pressure. If requests are slow while CPU time stays low, follow the request trace through D1, fetches and other waits first. Profiling is a diagnostic you run while investigating a workload.

1. Confirm the project account, Worker and version serving the affected traffic. The account needs Workers profiling enabled. Use the serving version ID during a rollout because `latest` selects the most recently created version. Upload source maps with the release so frames resolve to application code.
2. Capture while the operation is active. From the checkout, use the project's pinned cf invocation. For example, with pnpm and the Worker name and serving version set in the shell, this records five seconds of CPU samples:

   ```sh
   WORKER_ID=$(pnpm exec cf workers get "$WORKER_NAME" | jq -er '.id')
   pnpm exec cf workers versions profile "$VERSION_ID" \
     --worker-id "$WORKER_ID" --duration-ms 5000 \
     --profile-type cpu > worker-cpu.pprof
   ```

   Use the immutable Worker ID here. On PFA, the name returned error `10393` even while invocation logs confirmed traffic on the selected version. The ID yielded readable profiles. For allocations, use `--profile-type heap` and a separate output file. To target an active Durable Object instance running that version, supply both `--namespace-id` and `--actor-id`. The dashboard also supports captures under **Workers & Pages → Worker → Observability → Flamegraph**.
3. Read the widest stacks in the flame graph. CPU width reflects sampled CPU work. Heap width reflects sampled allocations during the window. A heap capture alone cannot establish that memory stays retained. A Worker capture samples one recently active isolate, so repeat under comparable traffic if the operation did not appear.
4. After a fix, repeat the same operation and capture duration. Record the versions, workload and change in CPU time per invocation or allocation volume. Check request-level metrics across the deployment before generalizing from one isolate. [Cloudflare’s guide to on-demand profiling](https://blog.cloudflare.com/workers-on-demand-profiling/)

### Attribute a slow request before changing code

Choose a user action and a success condition. For example, reduce time to display a financial summary while preserving the figures and keeping Worker CPU per request flat. Capture the following on the same requests.

| Measurement | Collection | What it tells you |
| --- | --- | --- |
| User elapsed time | Time from the click or request start to the named finish, such as body consumed or page usable | What the user waited for, including transport and client work |
| Worker CPU | Match the response’s Ray ID to the invocation log and read its CPU milliseconds and serving version | Compute attributed to this invocation. An on-demand CPU profile identifies functions sampled on one active isolate |
| Handler and dependency waits | Time authentication, the handler and important binding calls with spans or Server-Timing | Which dependency blocks the response. A parent span already contains its children. Parallel calls overlap |
| D1 execution and volume | Read statement metadata for SQL duration, rows read and written, attempts and serving location when available. Count returned rows separately | Database execution excludes network time. A small result can still require a large scan |

Record the ingress colo, the response’s `cf-placement`, and the D1 location separately. Use a harmless representative request, the same data, caller type, cache state and client. Warm each version and alternate them when possible. Group by serving version **and** execution location. Include an unchanged route using the same authentication and database. When placement differs, report the user latency by location and leave the code effect unresolved. [Worker placement](https://developers.cloudflare.com/workers/configuration/placement/) · [D1 result metadata](https://developers.cloudflare.com/d1/worker-api/return-object/)

Build the elapsed-time breakdown per request before summarizing it. Subtract only disjoint measured intervals. Report overlaps and unexplained time explicitly. Subtracting Worker CPU or summed SQL durations from client elapsed time cannot identify network latency. Invocation wall time can include background work after the response. Keep counts, medians and meaningful tails alongside the matched sample size. [Worker metrics](https://developers.cloudflare.com/workers/observability/metrics-and-analytics/)

For a sampled request, use the project CLI’s `o11y telemetry query`. Query the `traces` view over the probe’s time window, then query the `events` view with an equality filter on `$metadata.traceId`. Leave the dataset filter open for this second query. Invocation logs appear in `cloudflare-workers` and native spans in `otel`. Match the invocation’s Ray ID and version back to the probe. Inspect field names on the installed API version. Keep credentials, query parameters and returned records out of the report.

For D1, identify binding spans such as `d1_all` and `d1_first`. Clip their start and end timestamps to the root span, sort by start, merge overlaps, and sum the merged lengths. This gives elapsed time with at least one D1 call in flight. For each statement with metadata, read `source.cloudflare.d1.response` for SQL time, attempts, rows read and database colo. Report metadata coverage and check pagination before totaling a trace. One observed request had 61 binding calls spanning 835 ms of elapsed time, 3,685 ms if their overlapping durations were added, and 51 ms of SQL execution. CPU and outstanding I/O can overlap. [Telemetry query API](https://developers.cloudflare.com/api/resources/workers/subresources/observability/subresources/telemetry/methods/query/) · [Native span attributes](https://developers.cloudflare.com/workers/observability/traces/spans-and-attributes/)

To decide whether to keep a query rewrite, alternate the two read-only queries on the same fixture and compare complete application outputs. Check SQL duration, rows scanned, rows returned, and downstream Worker CPU. If the query moves a join or grouping step into JavaScript, replay each version’s result sets through its own decoder and handler under the same runtime. Report a local CPU replay separately from deployed CPU. For a change to concurrency, repeat with a fixed binding delay on both versions to expose serial round trips. The unchanged route shows how much noise remains.

A release decision should name the saved resource and any added cost. Multiply the per-request difference by observed route volume when estimating quota impact. Use the account’s actual allowances and rates for a money estimate. A reduction in reads may be worth keeping with flat CPU and latency. A measurable CPU increase calls for an explicit tradeoff. If a live profile cannot attach, retain invocation CPU measurements and leave function attribution open. [Production profiling](https://developers.cloudflare.com/workers/observability/profiling-in-production/)

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
