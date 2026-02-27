# Day 9 — Rate Limiting & API Resilience

## Executive Summary
Modern payment systems operate in high-volume, latency-sensitive environments where small traffic spikes can destabilize authorization, fraud scoring, or gateway layers. Rate limiting is an intentional protection mechanism that preserves system integrity, prevents cascading failures, and keeps financial transactions reliable under load.

This lab focuses on forcing rate-limiting behavior (HTTP 429), inspecting response headers, and documenting a retry model that avoids retry storms while supporting safe payment retries via idempotency.

---

## The Core Problem: Overload & the Thundering Herd
When traffic spikes suddenly (flash sales, subscription renewals, checkout bursts), multiple clients may attempt the same API request simultaneously.

This creates the **Thundering Herd Problem**, where:
- Identical requests flood the system
- Upstream dependencies saturate (datastores, auth, fraud engines)
- Latency climbs until timeouts and elevated error rates appear
- Retries amplify load and accelerate failure

In payments, the outcome can be checkout degradation, authorization failures, and duplicated attempts if retries are not governed.

---

## 429 Errors: A Protection Mechanism
A **429 Too Many Requests** response indicates the API is enforcing capacity thresholds. Correct behavior is to slow down and retry safely rather than immediately re-issuing the same request.

### Retry Model: Exponential Backoff + Jitter
A safe retry strategy:
- Honors server guidance when provided (ex: `Retry-After`)
- Applies **exponential backoff** (e.g., 1s, 2s, 4s, 8s…)
- Adds **jitter** (randomized delay) to prevent synchronized retry bursts
- Caps attempts to avoid indefinite loops and protect downstream systems

---

## Lab Reproduction: Triggering 429 and Capturing Evidence
### Endpoint Used
- `GET https://api.stripe.com/v1/balance` (rate-limit probe)

### Runner Settings
- Iterations: **2500**
- Delay: **0 ms**
- Concurrency: **Single Runner**

### Observed Result
- First 429 observed at approximately iteration: **1,829**
- Status code distribution: 200 (2,463) | 429 (37)
- Remaining requests resumed returning 200 after throttling window cleared

### Interpretation
The first rate-limit response occurred after sustained burst traffic with zero delay between requests.  
The presence of `Retry-After: 1` suggests a short rolling throttling window rather than a hard block.  
The API resumed returning 200 responses after the backoff interval elapsed, consistent with adaptive rate enforcement.
  
### Headers Captured (429 Response)
- `Retry-After`: **1**
- Request identifier: **req_9kH2JxT5LmYpQ**
- Additional relevant headers: **N/A**

### Sample 429 Response Body (Truncated)
```json
{
  "error": {
    "type": "rate_limit_error",
    "message": "Too many requests made to the API too quickly."
  }
}
```

### Payment Write Consideration
For write operations (e.g., PaymentIntent confirmation or charge creation), retries must only occur when an Idempotency-Key is supplied.  
Without idempotency, retrying after a 429 can result in duplicate financial transactions.  
Rate limiting and idempotency must be designed together in any payment architecture.
