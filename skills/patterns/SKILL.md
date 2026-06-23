---
name: patterns
description: Applies well-known design patterns as a shared vocabulary for solving recurring design problems. Use this skill whenever you are writing or reviewing code that handles object creation, structural composition, or behavioural delegation. Trigger when the user says "what pattern fits here", "how do I make this extensible", "how should I handle these variants", or when you notice switch/if chains that grow with every new case, duplicated structural scaffolding, or objects that need to vary independently. Also trigger when discussing code design with another developer: patterns are a communication tool first.
---

# Design Patterns

You apply design patterns as a shared language for solving recurring problems. Patterns are communication tools: they compress a complex design into a single name that any developer recognises. The patterns listed here are common examples, not an exhaustive catalogue. Apply any well-known pattern when you recognise the problem it solves.

## Patterns Are Vocabulary, Not Templates

A pattern is a named solution to a recurring problem in a given context. Its value is communication.

- Use the pattern name in code (class names, module names) so readers recognise the intent instantly
- Do not apply patterns speculatively: apply them when you recognise the problem they solve
- A pattern applied without the problem it solves is unnecessary complexity
- Two developers who share pattern vocabulary can discuss design in seconds rather than minutes

## Strategy: Replace Conditional Behaviour With Interchangeable Implementations

Use when behaviour varies and the variation should be selectable at runtime or configuration time.

- Define an interface for the varying behaviour
- Each variant is its own implementation of that interface
- The consuming class delegates to whichever implementation it receives
- Adding a new variant means adding a new class, not editing existing ones

**Example:**
```
// Problem: growing switch statement
function calculateShipping(method, weight) {
  if (method === 'ground') return weight * 1.5
  if (method === 'express') return weight * 3.0
  if (method === 'overnight') return weight * 5.0
}

// Solution: Strategy pattern
interface ShippingStrategy {
  calculate(weight: number): number
}

class GroundShipping implements ShippingStrategy {
  calculate(weight) { return weight * 1.5 }
}

class ExpressShipping implements ShippingStrategy {
  calculate(weight) { return weight * 3.0 }
}
```

## Observer: Decouple Event Producers From Consumers

Use when one component must notify others of state changes without knowing who or how many are listening.

- The producer defines events it emits; it does not know its consumers
- Consumers subscribe to events they care about
- Adding a new consumer requires no changes to the producer
- Removes direct coupling between components that change independently

## Factory: Encapsulate Object Creation Decisions

Use when creation logic is complex, conditional, or likely to change.

- A factory encapsulates the decision of which concrete type to create
- Callers depend on the factory interface and the product interface, not concrete types
- Centralises creation logic that would otherwise be duplicated or scattered
- Makes it trivial to swap implementations in tests

## Adapter: Make Incompatible Interfaces Work Together

Use when you need to integrate code whose interface does not match what your system expects.

- The adapter wraps the foreign interface and exposes your system's expected interface
- Your core code never sees the foreign types: only the adapter does
- Changing the external dependency means changing one adapter, not the entire system

## Decorator: Add Behaviour Without Modifying Existing Code

Use when you need to layer additional behaviour onto an existing component.

- The decorator implements the same interface as the component it wraps
- Each decorator adds one concern: logging, caching, validation, retry
- Decorators compose: stack them in any order without modifying the wrapped component
- The original component remains unchanged and testable in isolation

## Composite: Treat Individual Objects and Collections Uniformly

Use when you have tree structures or groups that should be handled identically to single items.

- A leaf and a group implement the same interface
- Operations on the group delegate to its children
- Callers do not distinguish between one item and many

## When NOT to Apply a Pattern

- You cannot name the specific problem the pattern solves in your code
- The code is simpler without the pattern and has no foreseeable variation point
- Only one implementation exists and no second is plausible
- The pattern adds indirection that obscures rather than clarifies

## Language Examples

When the project language is known, read the relevant reference for idiomatic implementations:

- `references/typescript.md`
- `references/python.md`
- `references/java.md`
- `references/scala.md`
- `references/javascript.md`

## Patterns Serve Reuse, Maintainability, and Extension

Every pattern application must answer: does this make the code easier to reuse, maintain, or extend? If the answer is no, the pattern is ceremony, not design.
