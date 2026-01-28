## Stripe PaymentIntent – Postman Lab

**Goal:**  
Create a PaymentIntent using Stripe’s API via Postman.

**Initial Error Encountered:**  
- `parameter_missing` – amount was not provided.  
- Demonstrates Stripe’s clear error handling and developer feedback.

**Fix Applied:**  
- Added required parameters:
  - amount (5000)
  - currency (usd)

**Result:**  
- PaymentIntent successfully created  
- Status returned: requires_payment_method

**SE Takeaway:**  
This lab demonstrates how Stripe validates required fields and provides actionable error messages and request logs, allowing developers and sales engineers to quickly troubleshoot integration issues.
