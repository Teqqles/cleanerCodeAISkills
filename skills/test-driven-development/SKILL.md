---
name: test-driven-development
description: Applies Test-Driven Development discipline when writing new code. Use this skill whenever the user asks you to implement a new function, class, service, or feature. Trigger when the user says "write a test for", "add TDD", "test first", or "write this with tests". Also trigger whenever you are about to write production code without a failing test. If the user asks you to implement something without mentioning tests, suggest the TDD approach and follow it unless they decline.
---

# Test-Driven Development

You follow Test-Driven Development. These rules apply to all languages and test frameworks.

## The Red-Green-Refactor Cycle

TDD is a discipline, not a testing style. Follow the cycle every time:

1. **Red**: Write a failing test that expresses the desired behaviour. No production code yet. The test fails for a logical reason, not a compile error.
2. **Green**: Write the minimum production code to pass the test. Nothing more.
3. **Refactor**: Improve the design of both test and implementation. All tests stay green throughout.

Writing tests after the fact is not TDD.

## Write the Test First

Before touching production code, write a test expressing what the code must do.

- The test is the specification; the implementation is the answer
- If you cannot write the test first, the interface is not clear yet: clarify before coding
- Tests written after implementation pass by construction; they do not constrain the design

## Implement Only What the Test Requires

- Write the simplest code that makes the current test pass
- "Just in case" code has no test to justify it: leave it out
- General solutions emerge from a series of specific tests, not from anticipating requirements

## Deterministic, Isolated, Fast Tests

- Tests produce the same result every run: no randomness, no time-of-day sensitivity, no flakiness
- Each test is independent: no shared mutable state, no ordered dependencies between tests
- Tests run in milliseconds: slow tests break the feedback loop TDD depends on
- Use fakes and in-memory implementations to eliminate I/O, network, and filesystem from unit tests

## Mock Only Stable Interfaces

- Never mock a concrete class: mock behaviour via an interface
- Do not mock third-party library types directly: wrap them in an abstraction you own, then fake that
- Hand-written fakes are preferred over mocking frameworks: they are explicit and easier to reason about
- A fake implements the contract faithfully; it does not just suppress calls

## Tests as Executable Specifications

Test names describe behaviour, not the method under test.

- Bad: `testCalculate()`
- Good: `calculate_returns_zero_when_input_is_empty()`
- Good: `throws_when_order_has_no_items()`

Follow Arrange-Act-Assert: set up context, perform one action, assert one outcome. One logical assertion per test makes failures obvious.

**Example:**
```
test("total price includes tax for taxable items") {
  // Arrange
  const cart = new Cart([new Item("book", 10.00, taxable: true)])

  // Act
  const total = cart.totalWithTax(taxRate: 0.2)

  // Assert
  assertEqual(total, 12.00)
}
```

## Language Examples

When the project language is known, read the relevant reference for Red-Green-Refactor cycle examples using the language's test framework:

- `references/typescript.md`: Jest
- `references/python.md`: pytest
- `references/java.md`: JUnit 5 + AssertJ
- `references/scala.md`: ScalaTest (AnyFlatSpec)
- `references/javascript.md`: Jest

## The Suite Enables Aggressive Refactoring

- Tests that require rewriting when you refactor were testing implementation detail, not behaviour: fix that first
- Delete tests that test implementation; update tests when requirements change; never leave tests commented out
- The goal is a suite you trust enough to refactor boldly
