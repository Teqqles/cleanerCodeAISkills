---
name: dependency-injection
description: Applies Dependency Injection principles when designing or reviewing code. Use this skill whenever you are writing classes, services, modules, or any component that has collaborators: databases, HTTP clients, loggers, clocks, configuration. Trigger when the user mentions services, constructors, wiring, coupling, testability, or says "how should I structure this", "how do I make this testable", or "this is hard to test". Also trigger when you notice concrete dependencies being instantiated inside business logic.
---

# Dependency Injection

You design systems using Dependency Injection. These principles apply to all languages and frameworks.

## Never Instantiate Concrete Dependencies in Business Logic

Code that creates its own dependencies cannot be tested in isolation and cannot be swapped without modification.

- Business logic classes never call `new ConcreteService()`, open database connections, read files, or call `DateTime.Now` directly
- A class that needs a collaborator receives it: it does not construct it
- Database connections, HTTP clients, clocks, loggers, file system access, and random number generators are all dependencies that must be injected

**Example:**
```
// Wrong: hidden dependencies, untestable
class OrderService {
  process(order) {
    const db = new SqlDatabase()
    const clock = Date.now()
    db.save({ ...order, at: clock })
  }
}

// Correct: dependencies declared, injectable, testable
class OrderService {
  constructor(db, clock) {
    this.db = db
    this.clock = clock
  }
  process(order) {
    this.db.save({ ...order, at: this.clock.now() })
  }
}
```

## Depend on Abstractions, Not Implementations

- Define interfaces (or abstract types) for all dependencies that cross boundaries
- Business logic depends on `IOrderRepository`, not `SqlOrderRepository`
- Keep interfaces narrow: one responsibility per interface
- The abstraction is owned by the consumer, not the provider

## Inject Through Constructors or Explicit Parameters

- Constructor injection is the default: all required dependencies appear in the constructor
- Use parameter injection for per-call dependencies (e.g., a cancellation token)
- Avoid property injection unless the framework requires it: it hides required dependencies
- The constructor signature declares what the class needs to function

## No Static State, Global Singletons, or Hidden Coupling

Static state and service locators make dependencies invisible, break testability, and cause subtle ordering bugs.

- No static mutable fields shared across calls
- No service locator pattern (`ServiceLocator.Get<T>()`) inside business logic
- No ambient statics from domain logic: `DateTime.UtcNow`, `Thread.CurrentPrincipal`, `HttpContext.Current`
- Pass these in as injected dependencies: `IClock`, `ICurrentUser`

## High-Level Policy Must Not Depend on Low-Level Detail

Dependency arrows always point inward. This is the Dependency Inversion Principle.

- Domain and application layers define the interfaces
- Infrastructure (databases, HTTP, file system) implements those interfaces
- Infrastructure depends on the domain: the domain never depends on infrastructure

## Every Component Must Be Replaceable in Tests

If you cannot substitute a fake without changing the class under test, the design is wrong.

- Write at least one hand-written fake per interface to validate the contract is testable
- Hand-written fakes (in-memory implementations) are preferred over mocking frameworks: they are explicit and easier to reason about
- A fake faithfully implements the contract

## Language Examples

When the project language is known, read the relevant reference for interface definitions, constructor injection patterns, in-memory fakes, and composition root examples:

- `references/typescript.md`
- `references/python.md`: uses `Protocol` for structural typing
- `references/java.md`: includes Spring composition root example
- `references/scala.md`: uses `trait` for interfaces
- `references/javascript.md`: uses JSDoc for lightweight contracts

## Composition at the Application Boundary

Wire up the object graph in one place: the application entry point (main, Program.cs, AppModule, etc.).

- Core logic is never aware of the DI container
- If a class knows about the container, it is in the wrong layer
- The composition root is the only place where concrete types appear together
