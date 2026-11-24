# Improve performance from a profile

Start with one operation someone actually performs, such as opening a page, syncing a feed, or calling an MCP tool. Write down what should determine its cost. For example, an incremental sync of 100 transactions should mostly depend on the incoming records and the reconciliation window. Five more years of stored history should add little work.

## Find where the work grows

1. **Capture the operation through its normal entry point.** Record the version, environment, workload and result. A page click can issue several requests. Count all of them. For a slow request, separate elapsed time, CPU time and waits. For an expensive query, distinguish statements, rows scanned, rows returned and rows written. For allocation pressure, capture allocations during the operation and measure retained memory separately if investigating a leak.
2. **Hold the request fixed and vary one input at a time.** Replay the same recent payload against short and long histories. Then hold history fixed and vary account count. For matching or grouping code, try repeated keys and mostly unique keys at the same row count. Exercise the case where inputs or links are absent. These comparisons reveal which work scales with history, fan-out, collisions or an unused branch.
3. **Test one explanation.** Follow a wide CPU stack back to its caller, or inspect the query plan behind a large scan. Count the operation you think is repeated. If a page loads transactions before checking whether it has a linked account, measure both linked and unlinked households. If a loop revisits used matches, preserve the input order and count or time the repeated-key case.
4. **Compare behavior and cost together.** Run old and new code on independent copies of the same fixture and compare the complete response. For writes, compare persisted rows, decisions and replay outcomes. Include a case where the tempting shortcut would fail, such as two transfers competing for a match across a reporting cutoff. Then repeat the same measurement. For timings, warm both versions consistently, report the statistic and run count, and keep a common-case control.
5. **Check the deployed operation.** Record the serving version and exercise the affected route. Compare request count, CPU, database work or latency over comparable traffic, including sample size. A production smoke test verifies behavior. A claim about fleet cost needs production volume and resource measurements.

## Worked examples

| Question | Controlled comparison | What changed |
| --- | --- | --- |
| Does an incremental sync scale with stored history? | Same 100 incoming transactions and one account, with one year and five years of synthetic history. Local D1 rows read grew from 4,908 to 20,802. | Looking up incoming IDs and indexing the reconciliation query brought the two runs to 1,432 and 1,426 reads. The balance decisions and write counts matched. |
| Does the distribution of values matter? | Same 7,300 synthetic transactions. With repeated transfer amounts, the matcher repeatedly visited deposits it had consumed. Unique amounts had no such backlog. | Remembering the consumed prefix brought median local cash-flow computation from about 52 ms to 1.5 ms across seven timed runs. The unique-amount control stayed near 1.5 ms. |

A useful handoff fits in a paragraph. Name the operation, fixture and environment. Give the before and after in the same units, the behavior checked, and the case that saw little benefit. Record how to replay the comparison. Keep a regression test for a concrete failure or a stable work bound when it would catch a recurrence. A machine-specific timing from one investigation belongs in the measurement report.

For runtime-specific collection, see [Worker profiling](cloudflare.md#profile-a-deployed-worker), [Cloudflare request attribution](cloudflare.md#attribute-a-slow-request-before-changing-code) and [MCP cost and performance](interfaces.md#measure-mcp-cost-and-performance).
