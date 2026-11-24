# Engineering handbook

Start with the task you are doing. Add a profile when the project uses that language, runtime or deployment model. A local CLI, a mobile app and a hosted service can all use the core guides. Each project's current requirements supply its product decisions.

| Task | Read |
| --- | --- |
| Interpret a requirement, preserve behavior, or propose a new standard | [Intent and decisions](guides/intent.md) |
| Choose a regression test, evaluate a gate, or change a budget | [Verification](guides/verification.md) |
| Import provider data, reconcile missing records, or replay writes | [Provider sync](guides/provider-sync.md) |
| Change sessions, credentials, permissions, or public request handling | [Security](guides/security.md) |
| Release software, migrate state, diagnose failures, or measure cost | [Operations](guides/operations.md) |
| Investigate slow operations, unnecessary reads, or allocation growth | [Improve performance from a profile](guides/profiling.md) |
| Measure MCP usage or reduce tool execution cost | [MCP cost and performance](guides/interfaces.md#measure-mcp-cost-and-performance) |
| Change a CLI, library, web, API, or MCP interface | [Interfaces](guides/interfaces.md) |

## Implementation profiles

| When the change involves | Use |
| --- | --- |
| TypeScript source gates, compilation, or package verification | [TypeScript and Abacus](profiles/typescript.md) |
| Browser journeys, staging, cookies, or response headers | [Hosted web services](profiles/web-services.md) |
| Develop, deploy or manage resources from a Cloudflare checkout | [Project-scoped cf CLI](profiles/cf-cli.md) |
| Workers, D1, bindings, tracing, or Cloudflare deployments | [Cloudflare operations](guides/cloudflare.md) |

For example, a Python CLI that changes its output format can use the interface and verification guides with its existing test runner. A TypeScript Worker changing a D1 query can combine the operations guide with the TypeScript and Cloudflare profiles.

## Adopt this handbook

In the project's entry-point instructions, link the relevant guide at a reviewed commit or tag. Put the link beside the local requirement and the command that exercises it. For example, a connector entry can point to the provider-sync guide, the project's deletion rule and its persistence suite. Review changes to the shared guidance before updating the pin.

The [coding standards gist](https://gist.github.com/tannerwj/defd95424231e010c3c3220cfe15f2fb) is the portable entry point.

## Give feedback

If an instruction affected a real decision, include a short note in the task handoff. The [feedback guide](guides/intent.md#leave-a-note-from-real-work) has an example and a template for suggestions worth tracking.
