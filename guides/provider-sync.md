# Change a provider sync

Use this when importing, refreshing or reconciling external records. Write down the source of truth, the identity of a record, what a complete response means, and which local decisions survive refresh.

## Data integrity

**Retryable creates.** Generate a stable operation identity before the first
network attempt and reuse it on offline replay. Verify response loss after the
server commits, not just failures before a request reaches it; replay must
return the same owned entity instead of creating a duplicate.

**Concurrent side effects.** Gate counters and derived history on the unique
identity of the winning insert or claim, not timestamp equality. Concurrent
requests can share a millisecond while using different retry keys. Verify
overlapping same-answer and opposing-answer requests, including the resulting
counters and history, rather than only sequential retries.

**Claims are atomic.** "Check then write" is two statements and a race. Use a
conditional write (`WHERE used = 0`, `WHERE NOT EXISTS …`) and act on whether
it changed a row.

## Worked example

A user approved a direct balance of 105 after a feed reported 100. The feed refreshes with 100 and a newer timestamp. In a product where approval stands until the feed amount changes, the displayed balance stays 105. In a second fixture the feed reports 110 and the new value wins. Exercise the real persistence and reload path with both fixtures. A calculator given the correct historical row cannot detect a loader that omitted that row.

Before interpreting absence as deletion, verify the requested window completed. Exercise a truncated page set and a complete empty window for the same tenant. Assert which existing records survive. For multi-tenant data, include a second tenant with the same provider record ID.

For a retry, classify the operation and failure first. A bounded provider read can honor throttling. Replaying a write needs an operation identity that survives a lost acknowledgement. Verify the committed state after replay. Record partial results and optional-step warnings alongside the run outcome.
