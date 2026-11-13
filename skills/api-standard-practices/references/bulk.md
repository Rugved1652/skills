# Bulk operations

Use for endpoints that process multiple resources in one request; ordinary CRUD does not need a batch abstraction.

- Bound item count, total request bytes, execution time, and worker concurrency. Reuse project limits; never start unbounded parallel requests or accept an unlimited batch because each item is small.
- Establish whether the operation is atomic or permits partial outcomes. For atomic database changes, validate and authorize every item, then enforce invariants within one transaction. External side effects need their own consistency strategy; a database rollback cannot undo them.
- For partial processing, use a contract that permits partial outcomes, such as a POST batch operation; never partially apply a single PATCH document. Give each input a stable correlation key or index and return its outcome, including failures and skipped items. Keep per-item results inside the shared envelope's `data`; its outer status describes the overall request. Document the HTTP outcome instead of hiding mixed results behind an “all succeeded” message.
- Apply resource, action, and tenant authorization to every item. Batch-level authentication is insufficient. Do not reveal inaccessible resources in item errors or summary counts.
- Define duplicate identifiers, conflicting operations on one resource, and ordering requirements. Reuse single-resource business logic without making internal HTTP calls.
- Make retries safe at the required batch or item boundary. Retrying only failed items must not repeat committed effects; handle uncertain outcomes through the existing idempotency mechanism. Reuse async jobs only when runtime warrants them.

**Verify:** mixed valid/forbidden items, oversize batches, atomic rollback or accurate partial results, and retry after a lost response without duplicate effects.
