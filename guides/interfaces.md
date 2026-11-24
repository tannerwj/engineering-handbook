# Design a shipped interface

Identify who consumes the interface and which observations they rely on. Preserve inputs, outputs and failure behavior for existing callers unless the project has approved a change.

For a CLI, exercise the installed command with an explicit working directory. Verify stdout, stderr, exit status and files changed for a success and an invalid input. For a library, exercise the exported API from a small consumer using the packaged artifact. Browser, API and MCP projects can use the sections below.

For an in-app form, CLI command or MCP tool that sends a note to the product maintainer, use [Product feedback](product-feedback.md).

## Frontend

**Rendering.** Choose rendering to match the interaction. A shareable content page can use server-rendered HTML. A stateful application may benefit from a client framework. Verify deep links, navigation, accessibility and asset updates through the shipped interface.

**SPA routing.** Hard navigation to a client-side route must not 404: 302
browsers to the hash route (or serve `index.html`); API clients keep JSON
404s. Test deep links with a hard load, not just in-app navigation.

**Forms.** Never mix native HTML validation (`required`, `pattern`) with custom
JS validation; the native layer blocks custom messages. Pick one.

## API design

**Operation IDs.** Every OpenAPI operation gets a unique `operationId` — agents expose operations as callable tools named by it; without one the tool is an anonymous method+path.

## MCP (Model Context Protocol)

**Large tool catalogs.** When measured tool-discovery or composition costs justify code mode, expose an `execute`
tool that runs model-written JS in an isolated sandbox with a typed proxy for
the real tools. Compare token use on representative calls.

**Cloudflare code-mode sandbox example** (for code execution):
- Validate auth *before* spinning up the isolate (fail closed; bad tokens
  can't burn compute).
- `globalOutbound: null`: zero network from generated code.
- Token lives in `ctx.props`, invisible across the RPC boundary; sandbox gets
  a `callTool` RPC, never the credential.
- No recursion (`execute` can't call `execute`).
- Choose CPU, wall time, input/output size, inner-call and concurrency limits from representative tasks. Test the limit and the first request over it.
- Every side effect runs through the existing permission-checked handlers.

**Public integrations serving both interactive and scripted clients.** Support API keys *and* OAuth (RFC 9728 protected
resource metadata at `/.well-known/oauth-protected-resource/<name>`, 401s
carry `WWW-Authenticate` with the metadata URL). Accept Client ID Metadata
Documents (claude.ai, Claude Code, VS Code) and dynamic client registration
(Cursor). Don't lock into just API keys; retrofitting OAuth is painful.

**Scopes.** Offer read-only credentials. A write attempt with one returns
403 `insufficient_scope` in `WWW-Authenticate` so clients can step up, and
`tools/list` hides tools the credential can't use.

**Only advertise what works.** Hide tools whose dependencies (bindings,
config) are missing instead of listing them and failing on call.

### Measure MCP cost and performance

Start with the question you need to answer. For example, can a client discover a tool and finish a two-read task within budget? Track a transport request, a top-level tool invocation, an inner operation and a quota charge as separate units. One execute call with two reads can produce one completion, two inner dispatches and three charges if both layers are metered. Publish the counting rule beside the metric.

- Emit a structured completion event at the tool handler boundary. Use the service name supplied by the runtime, a stable event name, tool or mode, elapsed milliseconds, a bounded outcome code, inner dispatch count and result bytes. Specify which validation failures reach the handler, what a caught inner failure means and whether bytes describe the payload or the protocol envelope.
- Use the existing telemetry sink. Keep generated code, prompts, error text, credentials, arguments and financial or other user data out of completion events. Set a retention and sampling budget across services. When sampling successes, record the sampling rate and account for it before comparing services. Keep exact quota enforcement and required audit records independent of sampled diagnostics.
- Aggregate by service and mode. Show invocation count, failure categories, median and p95 tool duration, operations per execute and payload size. Report the window, sample size and sampling method. A streaming HTTP connection can stay open long after a tool returns. Use provider CPU, database reads/writes, isolated executions and billing data to estimate cost.
- Profile common cheap requests as well as heavy calls. Listing two static tools can otherwise build hundreds of operation schemas. Initialize dependencies on first use, reuse them within that request and keep authorization in the operation handler. Share catalog metadata across requests only when every input that changes visibility or content is accounted for.
- For a tool that assembles several views, trace which inputs each view reads and which outputs the caller uses. Share identical reads within the request. For a comparison across dates, share the item list only when its membership is the same for both dates and resolve dated values at each cutoff. Count statements inside database batches and compare complete results just below and above a batch boundary.
- Exercise a large group and many groups separately. A long history for one account exposes repeated array copies. Many accounts expose a full-history scan per account. Compare complete results, including transfers whose other side belongs to a different account.
- Run the same discovery, composed read and controlled failure before and after an optimization. Verify results, access boundaries, quota charges and completion fields. Count the work removed (registry builds, serializations, database operations or provider calls). Report local evidence separately from production latency and savings.

A saved log query can answer these questions across a fleet. Add an in-product dashboard or a metrics tool when an identified consumer needs it and its access scope is defined.
