# Release and operate a change

Start with the artifact users receive and how it reaches them. For a library, install the packed release in a clean consumer. For a desktop app, upgrade an existing installation with representative saved state. For a service, exercise the release through its public entry point and include any scheduled work affected by the change.

## State and recovery

Identify which data outlives the release and who else reads or writes it. Verify that existing data survives the transition. If an older binary may run during rollout or rollback, exercise it against the migrated state. Identify a recovery point and verify how to use it before a destructive migration.

## Diagnose the user-visible outcome

Choose a question someone operating the product will need to answer. For a queued import, correlate acceptance, processing and terminal outcome with a non-secret operation ID. For a local CLI, report failures through its exit status and useful diagnostics. Exercise a controlled failure and find the evidence using the same access an operator or user has.

## Measure the limiting resource

Reproduce a representative workload, record its size and expected result, and compare runs under equivalent conditions. For a desktop indexer, test a directory large enough to reveal growth in peak memory. For a hosted query, compare elapsed time, work performed and requests as record count increases. Confirm equivalent output and ownership before interpreting an improvement.

Projects using Workers or D1 can follow [Cloudflare operations](cloudflare.md) for configuration, platform measurements and release commands.
