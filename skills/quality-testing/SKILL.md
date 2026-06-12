---
name: quality-testing
description: Applies the RIGHTBICEP heuristic to design thorough, high-quality tests. Use this skill whenever you are writing or reviewing tests: unit, integration, or end-to-end. Trigger when the user asks about test coverage, test quality, what to test, or says "am I testing this properly?", "write tests for this", "is this well tested?", or "review my tests". Also apply when you notice tests only cover the happy path: always add boundary, error, and inverse coverage. Run this skill after any code change to verify the suite is complete and passing.
---

# Quality Testing: RIGHTBICEP

You design tests using the RIGHTBICEP heuristic. This is the completeness standard for all projects and languages.

## The RIGHTBICEP Dimensions

Apply each dimension when designing tests for any non-trivial behaviour.

### R: Right
Verify correct output for valid, typical inputs.
- The happy path passes
- Results match expected values exactly
- Minimum viable: necessary but not sufficient

### I: Inverse
Verify that inverse operations undo each other.
- Write then read returns the same value
- Encode then decode restores the original
- Parse then serialize produces equivalent output
- Roundtrip tests catch subtle asymmetries

### G: Good
Verify typical use cases that represent real usage.
- Cover scenarios that occur most often in production
- Cover the normal input range a real caller would provide
- These tests define the expected contract

### H: Harmful
Verify handling of invalid, malicious, or adversarial inputs.
- Null values, empty collections, negative numbers, zero, max values
- Inputs designed to cause injection, overflow, or path traversal
- Syntactically valid but semantically wrong inputs
- The system degrades gracefully: it does not silently corrupt or crash

### T: Time
Verify time-dependent behaviour deterministically.
- Never use real clocks or wall time in tests: inject a controllable clock
- Test TTL expiration, scheduling, debouncing, and retry backoff with a fake clock
- Test at time boundaries: exactly at expiry, not just before or after

### B: Boundaries
Test limits, edges, and transitions.
- Off-by-one errors at collection boundaries (empty, one element, at-capacity)
- Numeric limits (zero, negative, INT_MAX, overflow)
- String boundaries (empty string, single character, maximum length)
- State transitions: from every valid state to every reachable next state

### I: Interfaces
Test interactions with external systems via fakes.
- Do not test against real databases, HTTP services, or filesystems in unit tests
- Use fakes that implement the same contract
- Keep integration tests (which may use real implementations) clearly separated
- An unconstrained fake hides bugs: the fake enforces the contract

### C: Cross-checks
Validate results using an independent mechanism.
- Compute the same value two ways and assert they agree
- Compare a fast implementation against a known-correct reference
- Verify state using a different read path than the one under test

### E: Error Conditions
Verify graceful handling of failures.
- Service unavailable, timeout, network partition
- Corrupt or unexpected input formats
- Missing configuration, missing files, insufficient permissions
- Errors produce the correct exception type, message, or error state: not a generic crash

### P: Performance
Verify efficiency under load.
- Bulk operations do not degrade algorithmically (O(n²) for a list operation is a bug)
- Set a concrete threshold: "process 10,000 records in under 100ms"
- Required for data pipelines, query builders, and batch processors; optional elsewhere

## Test Design Rules

**Test behaviour, not implementation.** Tests asserting on private internals break during refactoring. Test what the unit produces, not how it works.

**Avoid brittle tests.** Do not assert on fields irrelevant to the behaviour under test. A test that breaks when you rename an internal variable tests the wrong thing.

**One logical assertion per test.** Multiple assertions obscure which condition caused the failure.

**Test names describe behaviour.** `calculate_returns_zero_for_empty_input`, not `testCalculate`.

## Language Examples

When the project language is known, read the relevant reference for RIGHTBICEP examples using the language's test framework:

- `references/typescript.md`: Jest
- `references/python.md`: pytest
- `references/java.md`: JUnit 5 + AssertJ
- `references/scala.md`: ScalaTest (AnyFlatSpec)
- `references/javascript.md`: Jest

## Mandatory Quality Gate

Run the full test suite after any code change. Zero failures, zero warnings: no exceptions.

- Fix pre-existing failures: do not defer them
- "It was already broken" is not an acceptable state
- Warnings compound over time: fix them when you find them
