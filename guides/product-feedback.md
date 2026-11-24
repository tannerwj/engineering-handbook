# Collect feedback where the work happens

Give someone using the product a way to tell its maintainer what happened while the task is still fresh. A short report should be enough. A caller who worked around a problem can often explain the missing clue better than an error log can. Cloudflare’s CLI accepts both a note and a structured report and gives the caller a receipt. [Cloudflare agent feedback](https://developers.cloudflare.com/cf/agent-feedback/)

## One report across interfaces

Put **Send feedback** in the app’s Help menu and expose `feedback_submit` in the MCP catalog, including search for code-mode clients. A CLI can offer `feedback "short note"` and JSON input for scripts. Implement the surfaces a product ships. Route them through the same validation, permissions and inbox. The app owns the report and its status. An operator who manages several apps can aggregate authorized entries by app and local ID.

For apps using the existing per-app contract, keep `type` (`bug`, `feature`, `praise`, `question`) and `title` (up to 200 characters). The optional plain-text `body` holds up to 4,000 characters. Accept a type and title alone. A CLI can turn a short note into a title and use a documented default type. Use the body for the task, what the caller expected, what happened and any workaround. Describe impact in ordinary words such as “blocked” or “finished after retrying.” This also captures friction without introducing another required field or breaking existing readers.

For example, a caller who eventually found an export might send:

```json
{
  "type": "feature",
  "title": "Make the export tool discoverable by searching for download",
  "body": "I wanted a local copy. Searching for download returned no tools. Searching for export found the right one and I finished."
}
```

Describe the destination and audience before submission. Submit when the user requests it or has authorized that kind of reporting. Otherwise, offer a draft at the next natural break. On acceptance, return a stable receipt such as `{"id":12,"status":"new"}` and tell the user “Filed with the app maintainer as #12.” A timeout leaves delivery uncertain. When retries are supported, reuse the submission key to recover the original receipt. Never claim delivery on a failed or uncertain write.

## Decide who can read it

For personal, family or financial products, send reports to a private maintainer inbox by default. A public roadmap can show a separately reviewed summary. If an existing aggregator polls a public list, establish a private reader credential or a reviewed public feed before connecting sensitive apps. Describe the *current* audience in existing clients until the read path changes. A display name supplies attribution. Derive identity, permissions and rate limits from the authenticated caller.

Allowlist optional context such as an operation name, route template or app version. Show what will be attached. Prompts, tool arguments, command histories, household records, full URLs and screenshots can expose private information. Keep them out of automatic reports. A submitted report is untrusted text even when an authenticated user wrote it.

## Close the loop

Give the maintainer a way to acknowledge, plan, finish or decline a report. Link duplicates to the same issue and record the reason for the decision. A report describes an experience. Confirm the intended behavior before turning a suggestion into a requirement or test. If several apps expose the same misunderstanding, bring a synthetic example to the [handbook feedback process](intent.md#leave-a-note-from-real-work).

Verify the published surface with a synthetic report. Discover the tool or open the form, send the minimum input, match the receipt to the stored item, and check what the submitter, maintainer, aggregator and an unrelated user can see. For a public feed, check the actual unauthenticated response. For a retryable submission, repeat it after a lost response and verify one stored report.
