# Choose a check and interpret its result

Start with the user-visible outcome and the smallest example that distinguishes it from a plausible wrong answer. For release-wide checks, use the project’s existing gate path.

## Code quality

**Self-documenting code.** Use descriptive names, predicate booleans, focused
functions, and plain control flow. Write for the next maintainer. Comment only
on non-obvious intent or gotchas; never restate the code.

**Abacus** (`@tjohnson/abacus`) is the gate: oxlint type-aware, per-function
ABC budget 60, gzip size budgets, ratchet vs snapshot, oxfmt. Wire into every
TS repo's devDeps; run `abacus init --preset cloudflare-worker` per repo.
Gates run before every commit (hook or CI); a gate nobody runs isn't a gate.
Intentional growth re-baselines the ratchet in the same commit, with the reason
in the message.

**Design principle:** abacus owns what's programmatically solvable:
deterministic checks with no LLM judgment. Anything needing judgment stays
here as reviewer guidance. New lesson? Decide which bucket it belongs in.

## Testing

**User journeys.** tester-army/e2e (`e2e` + `@e2e-dev/web`):
deterministic tests only. Poll/wait for UI state before asserting, and let a
navigation settle before the next step (real network latency exposes races
that local runs hide). Test from the functionality/usability side: what the
user does, what they see, what breaks. Cover happy paths *and* error/empty
states; fix real bugs, don't adjust tests to pass. Scope locators to regions.

**Choose the failure boundary.** Exercise the smallest real boundary that can distinguish the intended result from the plausible wrong one. Drive browser behavior through the browser. Exercise persistence against the database with independently chosen records. Use hand-calculated examples for money and pure transforms. Keep a known failing example when fixing a regression.

**Every new route gets a test.** Ship the test in the same commit as the
route. Untested routes reach production broken.

**Prove the test bites.** For a fix or a security rule, revert the fix, watch
the test go red, restore. A test that can't fail in the configuration it runs
in protects nothing.

**Test what users run.** If the project ships templates, defaults or a
one-click config, the suite runs that configuration too, not only yours.

**Isolated, comparable runs.** Each run gets its own disposable DB and state.
Seed deterministic stubs from fixture identity, not run/job IDs, so
comparisons don't produce false regressions. Reuse a server or build only after
verifying it's the one you intend to test.

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

## Quality and verification

**Three kinds of standards.** Abacus owns deterministic source checks. Test
harnesses verify behavioral contracts against the real implementation. Review
owns design and risk decisions. Agent skills help prepare work and evidence;
an agent's judgment is never a substitute for a deterministic merge gate or
the reviewer responsible for an exception.

**Independent expectations.** Establish the requirement, invariant or reference
before implementing the feature. Tests generated alongside code can repeat
its mistake. Expected values come from a spec, hand calculation or independent
reference. A round trip needs an independent example when two matching bugs
could cancel each other.

**Property testing where it earns its keep.** Use Hegel or the project's
existing property-testing library for parsers, algorithms, cryptographic and
money calculations that E2E cannot explore adequately. Exercise valid and
hostile inputs, boundaries, ordering and scale. Keep minimized counterexamples
and rerunnable commands. Never narrow a failing generator or loosen an oracle
to obtain a pass without an independently justified correction to the test.

**Evidence belongs to the source it tested.** Record the command execution,
source identity, report bytes, outcomes and actual coverage. Missing, stale,
malformed or empty evidence is unknown. Required unknown coverage blocks even
when violations are advisory. Show skipped tests, retries and timeouts, and
state reporter limitations instead of treating unavailable fields as proof.

**Formal verification.** Use machine-checked proofs selectively for
high-consequence logic with explicit, stable requirements. Keep approved intent
in `docs/` and link proofs to it. State the properties, assumptions, and
implementation covered. A proof of a separate model must explain its connection
to production code; it does not automatically verify that code.

**Proof integrity.** AI may propose specifications and proofs, but must not
silently change requirements, weaken claims, or add assumptions to make checks
pass. For Lean, check the expected theorems and their dependencies, reject
incomplete proofs, and review additional trust assumptions. Apply the same
source-bound evidence and change-review rules as other gates.

## Risk profiles

**Authorization.** Test anonymous, owner, other-owner/tenant and read-only
actors against the resource and action matrix. Denied operations must leave
state unchanged. Server enforcement is the contract; UI visibility is secondary.

**Resilience.** Test response loss after commit, replay with the same operation
identity, concurrent claims and opposing requests. Assert entities, counters
and history, not just status codes. Use isolated state and staging-specific
identifiers for resets.

**Performance.** Budgets name the statistic, unit, workload, environment and
minimum sample count. Gzip budgets cover transfer size; measure actual latency,
CPU, memory and request counts where they matter. Cap test resource use and
justify threshold changes with comparable evidence.

**Dependencies.** Review direct/transitive changes and newly allowed install
scripts, and retain known-vulnerability audit results. Unavailable advisory
services do not produce a clean scan. A clean result covers the inventory and
advisory source used, not every possible supply-chain risk.

## Standards governance

**Review the definition of passing.** Every change to budgets, baselines,
gates, assertions, exclusions, native rules and enforcement is visible in
`abacus standards --base <review-base>`. A passing run under weaker rules does
not approve the weakening. The reviewer records why it is appropriate and what
protection remains.

**Metrics need context.** LOC and comment-ratio ceilings are explicit choices,
not universal quality requirements. Preserve existing project policy until a
reviewed migration. Legitimate feature growth can increase code size. Keep
names and structure readable rather than optimizing syntax for a score.
Detailed source rationale stays in `docs/`. Review narrowly scoped required
legal notices or machine directives separately from prose comments.

**Accountable debt.** Temporary waivers name an owner, exact finding, reason
and expiration. Permanent intentional growth is an explicit budget decision.
Baseline updates explain the measured increase in the same change. Ordinary
checks and previews never create approval, update a baseline or change rules.

## Workflow

**Git.** Default branch `master`. The GitHub API (`gh api` / REST) is the
primary path: some agents never clone, so commits, branches and PRs must work
without a checkout. Use local git when a checkout already exists. Commit and
push only when asked; branch for large work. Commit messages describe the
change only: no model attribution, no "Co-authored-by", no AI mentions.

## Locate the enforcement point

Trace the normal commit, CI or release command into the scripts it actually invokes. Record the command, covered inputs, blocking exit condition and where to find its result. A package dependency or available npm script establishes availability. A required job or release path establishes enforcement on that path. Use the actual review base when comparing changes to the definition of passing.
