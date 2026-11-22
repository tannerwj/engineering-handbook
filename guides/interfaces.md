# Design a shipped interface

Identify who consumes the interface and which observations they rely on. Preserve inputs, outputs and failure behavior for existing callers unless the project has approved a change.

For a CLI, exercise the installed command with an explicit working directory. Verify stdout, stderr, exit status and files changed for a success and an invalid input. For a library, exercise the exported API from a small consumer using the packaged artifact. Browser, API and MCP projects can use the sections below.

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
- Resource caps: 20s CPU / 25s wall / 200KB code / 500 subrequests.
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

**Metrics from day one.** Track every tool call: timestamp, tool name,
duration, outcome and, when per-credential reporting is needed, a non-secret credential record ID. Expose a `metrics_summary` tool (per-tool
calls, p50/p95 latency, top errors, execute composition) *and* a human
dashboard. The "most-chained tools" and "zero-result searches" signals tell
you where the catalog needs work.
