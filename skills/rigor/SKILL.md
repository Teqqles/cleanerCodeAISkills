---
name: rigor
description: Applies rigorous engineering discipline: explicit error handling, failure-first design, observability, and production-readiness. Use this skill whenever you are writing code that runs in production, handles errors, calls external services, or needs to be diagnosed when things go wrong. Trigger when the user says "make this production-ready", "how should I handle errors here", "this needs to be reliable", "add logging", or "what could go wrong". Also trigger when you notice unhandled failure paths, swallowed exceptions, hardcoded config, or missing observability in code you are writing or reviewing.
---

# Engineering Rigor

You apply rigorous engineering discipline. These practices apply to all production code regardless of language or scale.

## Validate Assumptions With Tests or Proofs

Unvalidated assumptions become bugs. Code built on unverified assumptions is built on sand.

- Every non-trivial assumption about inputs, state, or system behaviour is covered by a test
- Preconditions, invariants, and range constraints are explicit: asserted, encoded in the type system, or covered by a test
- "I think this will always be non-null" is a bug waiting to happen: prove it or guard against it

## Handle Errors Explicitly and Predictably

Every operation that can fail has a defined failure path.

- Return typed errors, throw typed exceptions, or use result types: errors never bubble silently
- Error messages are actionable: what went wrong, where, and what to do about it
- Catching and swallowing exceptions is almost always wrong: log, rethrow, or handle explicitly
- Generic catch-all handlers are a last resort, not a default

**Example:**
```
// Swallowed: bug hidden from the operator
try {
  processPayment(order)
} catch (e) {}

// Explicit: failure surfaced and actionable
try {
  processPayment(order)
} catch (PaymentGatewayError e) {
  logger.error("Payment failed for order {}", order.id, e)
  throw new OrderProcessingError("Payment declined", order.id, e)
}
```

## Design for Failure, Not Just Success

For every external call (network, filesystem, database), ask: what happens when this fails?

- Operations that can fail transiently have retry, circuit-breaker, or fallback logic
- The system tolerates partial failure without corrupting state
- Test failure paths with the same care as success paths
- Assume the network is unreliable, disks are finite, and services will be slow

## Avoid Premature Optimisation: But Choose Correct Algorithms

- Do not optimise until you measure a performance problem
- Choose appropriate data structures and algorithms from the start: O(n²) over a growing list is a design decision, not an optimisation
- Measure before optimising; assumptions about what is slow are usually wrong
- Document performance requirements as concrete thresholds: "process 10,000 records in under 100ms"

## Production-Ready From the Start

Code written to ship is observable and correctly configured.

- Log meaningful events: significant state transitions, external calls, failures with context
- Expose health checks for infrastructure monitoring
- Configuration comes from the environment: never hardcoded
- Secrets never appear in code or logs
- The application starts cleanly, fails fast on misconfiguration, and shuts down gracefully

## Observable and Diagnosable in Production

Errors are diagnosable from logs alone, without reading the source code.

- Log at the right level: `error` for failures requiring action, `info` for significant transitions, `debug` for diagnostic detail
- Structure logs for filtering and querying: consistent field names, machine-readable format
- Errors include enough context: what was the input, what was the state, what was expected
- Never log sensitive data: passwords, tokens, PII

## Language Examples

When the project language is known, read the relevant reference for typed error results, precondition enforcement, structured logging, and environment-driven config:

- `references/typescript.md`: uses discriminated union for typed results
- `references/python.md`: uses dataclasses for result types
- `references/java.md`: uses sealed interfaces for typed results
- `references/scala.md`: uses sealed traits and `Either`
- `references/javascript.md`: uses result objects

## Every Change Improves Clarity, Correctness, or Capability

Before committing, ask: does this change make the code clearer, more correct, or more capable?

- Neutral changes add noise to history without value: don't make them
- This standard applies to refactors, features, and fixes equally
