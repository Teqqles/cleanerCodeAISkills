---
name: api-design
description: Applies principled API design: predictable, minimal, explicit, and testable interfaces. Use this skill whenever the user is designing a public API, library interface, HTTP endpoint, module boundary, or CLI interface. Trigger when the user says "how should I design this API", "what should this function signature look like", "how do I make this interface clean", or "review this API". Also trigger when you notice surprising behaviour, hidden side effects, leaking implementation details, or overly broad parameter lists in any interface you are writing or reviewing.
---

# API Design

You design APIs that are predictable, minimal, and expressive. These principles apply to all public interfaces: library APIs, HTTP endpoints, internal module boundaries, and CLI interfaces.

## Predictable Behaviour: No Surprises

Developers form a mental model of an API and act on it. Behaviour that violates expectations causes bugs that are hard to find.

- Similar operations work in similar ways: consistent naming, consistent return types, consistent error signalling
- The same inputs produce the same outputs (or the same failure modes)
- Never silently swallow errors or return null/empty when the contract requires a value
- Never mutate state as a side effect of a read operation

## Minimal Surface Area

Every public symbol is a commitment to maintain. Expose only what callers need.

- Default to private/internal; make things public when there is a concrete use case
- Fewer parameters are better: if a function takes more than three or four, consider a parameter object
- Remove parameters that do not affect behaviour; they confuse callers and complicate testing
- A minimal API is easier to learn, easier to test, and easier to evolve

## Explicit Over Implicit

Callers understand what a call does from the call site alone.

- Never rely on global state, ambient context, or implicit ordering to make the API work
- Required inputs appear in the function signature: not in environment variables or module-level state
- Enforce preconditions explicitly (validate and throw or return error) rather than failing mysteriously later

**Example:**
```
// Implicit: hidden dependencies
function processOrder() {
  const user = getCurrentUser()
  const config = globalConfig.tax
  ...
}

// Explicit: everything declared
function processOrder(order, user, taxConfig) { ... }
```

## Design Around User Intent, Not Internal Constraints

The API expresses what the caller wants to achieve, not how the implementation is structured.

- If the caller must make three calls in a specific order to accomplish one task, wrap that in a single operation
- Do not leak implementation details (internal IDs, framework types, database schemas) into the API contract
- Use domain language: the caller's vocabulary is the API's vocabulary

## Stable, Versioned Public APIs

Once an API is public, breaking changes require a version increment or a deprecation period.

- Add new capabilities through additive changes (new endpoints, new optional parameters)
- Deprecate before removal; announce removal well in advance
- Every public function signature is a contract

## Document Behaviour Through Examples and Tests

- Working examples are the primary documentation: code that compiles and runs
- Tests are specifications: they demonstrate expected usage and expected failure modes
- Inline documentation describes the contract, not the implementation
- Every public function has at least one example showing the common use case

## Language Examples

When the project language is known, read the relevant reference for explicit inputs, parameter objects, contract documentation, and versioning patterns:

- `references/typescript.md`
- `references/python.md`: uses dataclasses for request objects
- `references/java.md`: uses records for request objects
- `references/scala.md`: uses case classes; implicits section shows hidden-dependency anti-pattern
- `references/javascript.md`: uses options objects for minimal parameter surface

## Easy to Mock, Fake, or Stub

Every API crossing a boundary (I/O, network, time, randomness) sits behind an interface.

- The interface is narrow enough that a fake can be written in a few lines
- If a caller cannot use the API in a test without real infrastructure, the design needs work
- See the `dependency-injection` skill for how to structure injectable interfaces
