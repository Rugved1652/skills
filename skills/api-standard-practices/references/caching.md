# Caching

Use when implementing cache behavior or fixing stale/leaked results. Choose HTTP caching, application caching, or both deliberately; HTTP headers do not configure an application cache. Do not introduce cache infrastructure without a concrete need.

## Scope and freshness

- Establish acceptable staleness and what invalidates a result before selecting a TTL. Key application entries by resource/query, pagination, representation version, and every relevant tenant/user/permission dimension. Authorize access even on a cache hit; account for permission revocation.
- Invalidate or version affected application entries after committed writes, including list/search results. Consider concurrent readers repopulating stale entries and failure recovery when consistency requirements demand it. TTL alone does not guarantee immediate read-after-write consistency.
- Bound cache size and loading work. Choose outage behavior explicitly; a cache used for correctness or security cannot silently become an optional optimization.

## HTTP behavior

- `Cache-Control: no-store` prohibits storing the response; `no-cache` permits storage but requires successful revalidation before reuse; unqualified `private` prohibits shared-cache storage while allowing a private cache. These directives do not replace authorization. [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111.html#section-5.2.2).
- Set freshness deliberately. Do not mark personalized responses `public` or enable shared caching without a proven isolation contract. Use `Vary` for request headers that select representations; it does not establish access control.
- Generate validators for the selected representation. After normal authorization, evaluate conditional GET/HEAD requests using `ETag`/`If-None-Match` or supported modification dates. Return bodyless `304` only when the condition warrants it, retaining required validator/cache headers. Do not wrap it in the JSON envelope. [HTTP conditional requests](https://www.rfc-editor.org/rfc/rfc9110.html#section-13).

Verify tenant isolation, permission revocation, query-key differences, invalidation after commit versus rollback, concurrent stale fills, and matching/nonmatching validators with bodyless `304` responses.
