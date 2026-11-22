# Engineering handbook

Shared coding standards for Tanner's projects. Start with the row for the change you are making. Each project's current requirements supply its product decisions.

| Task | Read |
| --- | --- |
| Interpret a requirement, preserve behavior, or propose a new standard | [Intent and decisions](guides/intent.md) |
| Choose a regression test, evaluate a gate, or change a budget | [Verification](guides/verification.md) |
| Import provider data, reconcile missing records, or replay writes | [Provider sync](guides/provider-sync.md) |
| Change sessions, credentials, permissions, or public request handling | [Security](guides/security.md) |
| Change a Worker, D1 query, deployment, trace, or portable install | [Cloudflare operations](guides/cloudflare.md) |
| Change a web, API, or MCP interface | [Interfaces](guides/interfaces.md) |

## Adopt this handbook

In the project's entry-point instructions, link the relevant guide at a reviewed commit or tag. Put the link beside the local requirement and the command that exercises it. For example, a connector entry can point to the provider-sync guide, the project's deletion rule and its persistence suite. Review changes to the shared guidance before updating the pin.

[Abacus](https://github.com/tannerwj/abacus) supplies deterministic checks. The [coding standards gist](https://gist.github.com/tannerwj/defd95424231e010c3c3220cfe15f2fb) is the portable entry point.
