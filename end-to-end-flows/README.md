# Day 7 – End-to-End Payment Flows

## Flow 1: Successful Card Payment
1. Client creates PaymentIntent
2. Stripe validates and processes payment
3. payment_intent.created event emitted
4. Webhook delivered to merchant system
5. Payment succeeds
6. payment_intent.succeeded event delivered

## Flow 2: Payment with Fraud Review
1. PaymentIntent created
2. Risk signals evaluated
3. Transaction flagged for review
4. Additional verification required
5. Final decision event emitted
