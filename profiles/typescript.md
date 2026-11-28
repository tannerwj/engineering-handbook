# TypeScript projects

Use the project's package manager, compiler projects, lockfile and normal build entry point. A shared library, browser application and Worker can have different compilation targets. Trace the source and shipped artifacts covered by each check.

## Source gates

Use Abacus for type-aware oxlint, oxfmt, per-function ABC complexity, ratchets and configured size budgets. For new projects choose the matching preset, such as `typescript`, `vite-spa` or `cloudflare-worker`. The usual starting ABC budget is 60. Read the repository's configured budgets before changing code. Name the measured increase and reason when reviewing intentional baseline growth. [Abacus setup](https://github.com/tannerwj/abacus/blob/master/docs/getting-started.md)

Run the applicable checks through the normal commit, CI or release path. Compare standards changes against the actual review base with `abacus standards --base <revision>`. Use `--fail-on weakening` where the project requires that comparison to block. Use `abacus secrets` for the configured gitleaks scan. [Standards comparison](https://github.com/tannerwj/abacus/blob/master/docs/standards-changes.md)

## Pick the runtime evidence

For parsers or pure calculations with a broad input space, use Hegel or the existing property-test library and retain minimized failing cases. For browser journeys using tester-army/e2e, run the shipped interface and wait for observable state. For a published package, exercise its real exports from a clean consumer install. A passing typecheck alone cannot detect missing files in the published package.

For Worker response headers, use a common Hono middleware or a wrapper on the exported fetch handler. Document required local secrets by name in `.dev.vars.example`. For typed failures and stateful frontend workflows, see [Effect and Effect UI](effect.md). Worker observability, D1, and infrastructure trials are in [Cloudflare operations](../guides/cloudflare.md).
