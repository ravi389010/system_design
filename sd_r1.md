# Rule 1: Every Remote Call Needs a Timeout

## Why Does This Rule Matter?

A remote call is not like a local function call.

When your service calls a database, payment gateway, inventory API, or another microservice, it crosses a network boundary. That means the response can be slow, lost, partially delivered, or never arrive at all.

**Without a timeout, the calling service keeps waiting.**

## Real-World Scenario: E-Commerce Checkout During a Flash Sale

Imagine an e-commerce checkout service calling a payment gateway during a flash sale.

1. **A customer submits a payment.**
2. **Checkout sends the request to the payment gateway.**
3. **The gateway becomes slow because of high traffic.**
4. **Checkout waits for the response.**
5. **More customers arrive, creating more waiting requests.**
6. **Threads, connections, memory, and worker capacity become occupied.**
7. **Even healthy requests begin to fail.**

**The original problem was a slow payment gateway. The real incident becomes a checkout outage.**

That is why every remote call needs a timeout.

## What Is a Timeout?

**A timeout defines the maximum time your service is willing to wait for a dependency.**

For example, consider a checkout service with a total response-time target of 4 seconds.

| Operation | Allocated Time |
|---|---:|
| Authentication | 500 ms |
| Inventory validation | 700 ms |
| Payment authorization | 2 seconds |
| Processing and response overhead | 800 ms |
| **Total** | **4 seconds** |

**If the payment gateway has not responded within those 2 seconds, checkout stops waiting.**

It records the timeout, releases the thread and connection, and returns a controlled response such as:

> Your payment is taking longer than expected. Please try again shortly.

## Why Are Timeouts Important?

**This is not merely about speed. It is capacity protection.**

**A well-designed timeout prevents one unhealthy dependency from consuming all available resources in the calling service.**

## Practical Principles for Implementing Timeouts

1. **Set timeouts intentionally.** They should come from the end-to-end request deadline, not a random default value.

2. **Use different timeouts for different operations.** A product search and a payment authorization have different latency expectations.

3. **Set connection and response timeouts.** Connecting to a service and waiting for its response are separate failure modes.

4. **Do not set timeouts so high that they become ineffective.** A 60-second timeout for a 3-second user journey is not protection.

5. **Do not set them so low that healthy requests fail unnecessarily.** Use real latency data and leave a sensible margin.

6. **Measure timeout rates.** A rising timeout rate is often an early warning that a dependency is degrading.

## Important Consideration: Timeout Does Not Mean Cancellation

**A timeout does not guarantee that the remote system stopped processing the request.**

The payment gateway might complete the charge after checkout has timed out.

**This is why timeouts must work together with idempotency, retries, and reconciliation—especially for financial operations.**

For example:

- **Idempotency:** Prevent duplicate charges when the same payment request is submitted again.
- **Controlled retries:** Retry only when the operation and failure conditions make it safe.
- **Reconciliation:** Verify the actual payment status when the outcome is uncertain.

## Final Takeaway

**A timeout is a boundary.**

**It says: this dependency may be slow, but it is not allowed to take down the rest of the platform.**
