# Webhooks

Use for inbound events or outbound delivery; reuse the project's existing event infrastructure.

## Receive

- Follow the provider's documented verification, acknowledgement, timeout, and retry contract. Verify signatures before trusting payloads; preserve exact raw bytes when required and use the supported verification library. Apply documented timestamp tolerance and key rotation; never log signing secrets. [Stripe example](https://docs.stripe.com/webhooks).
- Validate supported event types and bind the authenticated sender to the correct account/tenant. Expect duplicates and out-of-order events; use scoped event identity and transactional deduplication with effects, or another durable recovery design. A timestamp check alone does not prevent duplicate processing.
- Acknowledge only after successful handling or durable acceptance for later processing. An in-memory task is not durable acceptance. For duplicate events already accepted, return the contract's success response. Do not assume every provider retries every failure or guarantees ordering; reconcile authoritative state when event order matters.

## Deliver

- Authenticate deliveries using the agreed protocol over HTTPS. Preserve event identity across attempts; retries may have fresh delivery timestamps/signatures. Persist delivery state and coordinate event publication with committed business changes when lost events would violate requirements.
- Bound attempts, elapsed retry time, concurrency, and request duration. Use backoff with jitter, honor applicable retry hints, and distinguish transient failures from permanent rejection. Record exhausted deliveries for controlled recovery; do not promise exactly-once effects.
- Treat configurable destinations as SSRF inputs. Allowlist destinations when possible; otherwise reject prohibited schemes/ports and private, loopback, link-local, or metadata targets. Validate resolved addresses and enforce them at connection time to resist DNS rebinding. Disable redirects or revalidate every hop. [OWASP SSRF guidance](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

Verify invalid signatures, concurrent duplicates, reordered events, failed durable writes, retry exhaustion, and redirect/DNS attempts toward internal services.
