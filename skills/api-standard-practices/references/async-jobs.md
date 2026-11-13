# Async jobs

Use when requested work needs background execution or exceeds a practical request lifetime; reuse existing workers and storage.

- Validate input, authorize the action, and enforce admission limits before accepting work. For durable workflows, persist enough information to recover execution before returning `202`. Close the database/dispatch failure gap with the existing durable queue or transactional dispatch mechanism; an in-process background task alone is insufficient.
- Return a job identifier and authorized status URL. `202` signals acceptance, not successful completion; describe the current state and monitoring path as specified by [HTTP semantics](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.3.3).
- Scope status, results, and cancellation to authorized callers. Carry trusted execution identity and tenant scope; apply the workflow's authorization policy at execution time without trusting client-supplied ownership.
- Persist queued/running/terminal states and results consistently. A successful status lookup can return `200` with `data.state: "failed"`; that reports a retrieved job, not a failed HTTP lookup. Report cancellation only when effective, and never invent completion percentages or claim rollback of completed effects.
- Expect redelivery and crashes. Make consequential effects safe to repeat with existing idempotency/atomic mechanisms; worker locks alone do not prevent duplicate effects after a crash. Retry transient failures with bounded attempts, backoff, and deadlines. Stop permanent failures and expose safe failure information.
- Define result retention and recover abandoned running jobs according to existing worker semantics; avoid jobs remaining “running” forever.

**Verify:** crash before/after dispatch, worker redelivery after an effect, cross-tenant status access, exhausted retries, and cancellation racing completion.
