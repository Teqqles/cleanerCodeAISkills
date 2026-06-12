---
name: maintainable-architecture
description: Applies maintainable architecture principles when designing or reviewing systems. Use this skill whenever the user asks how to structure a project, organise code, choose between inheritance and composition, or separate concerns. Trigger on "how should I architect this", "where does this code belong", "is this a good structure", or "I'm finding this hard to change". Also trigger when you notice business logic mixed with framework code, infrastructure types leaking into domain types, or modules with too many responsibilities.
---

# Maintainable Architecture

You design systems with maintainable architecture principles. These apply to all languages, frameworks, and project scales.

## Separate Domain Logic from Infrastructure

Business rules and infrastructure change at different rates and for different reasons. Mixed together, they produce code that is hard to test and impossible to evolve independently.

- Domain logic (calculations, validations, business rules) lives in its own layer with no framework imports
- Infrastructure (databases, HTTP clients, file system, queues) implements interfaces defined in the domain layer
- Changing the database technology must not require changing any business logic
- The domain layer is testable without starting a server, connecting a database, or touching the network

## No Infrastructure Leaking Into Core Logic

- ORM entities, HTTP request/response types, and framework annotations do not appear in domain types
- Map at the boundary: convert infrastructure types to domain types at the edge of the system
- Domain events, value objects, and aggregates are plain types: no persistence annotations, no serialisation attributes, no framework base classes

## Framework-Agnostic Business Rules

- Business rules are expressible as pure functions or simple classes that a test invokes directly
- If you cannot test a business rule without starting the web framework, the rule is in the wrong place
- Routing, middleware, request parsing, and response formatting are framework concerns: they stay out of the domain

## Small, Cohesive, Loosely Coupled Modules

- A module has one reason to change
- Group things that change together; separate things that change independently
- Many small modules with clear interfaces beat a few large ones with broad responsibilities
- A module that depends on many others is a design smell: split it or invert the dependency

## Open for Extension, Closed for Modification

Add behaviour without editing existing, working code.

- Use interfaces, strategy patterns, and composition over if/switch chains that grow with every new case
- When you reach for `else if` to handle a new variant, ask whether a new implementation of an existing interface is cleaner

## Composition Over Inheritance

- Assemble behaviour from small, focused objects over deep inheritance hierarchies
- Use inheritance only when the subtype is a true specialisation of the supertype ("is-a"), not when it just reuses code
- Delegating to a composed object is almost always clearer than overriding a method

**Example:**
```
// Inheritance: fragile hierarchy
class PremiumUser extends User {
  getDiscount() { return 0.2 }
}

// Composition: flexible, testable
class User {
  constructor(discountPolicy) {
    this.discountPolicy = discountPolicy
  }
  getDiscount() { return this.discountPolicy.calculate() }
}
```

## Language Examples

When the project language is known, read the relevant reference for layer separation diagrams, domain type patterns, boundary mapping, and composition over inheritance examples:

- `references/typescript.md`
- `references/python.md`: uses frozen dataclasses for domain types
- `references/java.md`: uses records for domain types
- `references/scala.md`: uses case classes and traits
- `references/javascript.md`

## Design for Testing, Refactoring, and Evolution

- Hard to test means wrong boundaries: fix the structure
- Make the common change easy; make the uncommon change possible
- Do not optimise for hypothetical future requirements: optimise so requirements can be added when they arrive
- Module boundaries accumulate coupling over time: review them regularly
