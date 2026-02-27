# Day 10 — Timeouts, Retries, and Exactly-Once PaymentIntent

## Objective
Demonstrate a retry-safe payment write pattern using idempotency keys, plus a documented retry policy for transient failures (429, 5xx, network timeouts).

---

## Implementation
### Requests
1. `POST https://api.stripe.com/v1/payment_intents`
   - Header: `Idempotency-Key: {{idempotency_key}}`
   - Captures: `payment_intent_id` + request identifier (when present)
2. `GET https://api.stripe.com/v1/payment_intents/{{payment_intent_id}}`

### State + Correlation
Persisted to environment for auditability and chaining:
- `idempotency_key`
- `payment_intent_id`
- request identifier header (when present)

---

## Evidence: Exactly-Once Intent
Repeated writes using the same `Idempotency-Key` returned the same PaymentIntent id.

- First response `payment_intent_id`: `pi_3T5Jr0ADjpwsX7Ik0f87oZgW`
- Second response `payment_intent_id`: `pi_3T5Jr0ADjpwsX7Ik0f87oZgW` (same)

Key behavior:
- Same Idempotency-Key → same object id (deduped write)
- New Idempotency-Key → new object id (new write)

---

## Retry Policy
Retryable conditions:
- 429: honor `Retry-After` when present; otherwise exponential backoff + jitter
- 500/502/503/504: exponential backoff + jitter
- network timeout/no response: retry only for idempotent operations

Non-retryable conditions:
- 400/401/403: fix request/auth; do not retry
- validation errors: fix payload; do not retry

---

## Payment Safety Note
Retries for payment write operations are only safe when idempotency keys are enforced. This prevents duplicate financial transactions during transient failures while allowing controlled recovery.
