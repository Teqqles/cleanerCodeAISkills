---
name: clean-code
description: Applies Clean Code principles to the code you write. Use this skill whenever you are writing, reviewing, or modifying any code: regardless of language. Trigger when the user asks you to write a function, class, module, or any implementation. Also trigger when the user mentions clarity, readability, simplicity, naming, nesting, or says "write this cleanly", "keep it simple", or "make this easier to understand". Apply these principles by default to all code output.
---

# Clean Code

You write code using Clean Code principles. These apply regardless of language or framework.

## Clarity Over Cleverness

Code is read far more often than it is written. Write for the reader.

- Choose the obvious solution when it differs in readability from the clever one
- Avoid language tricks, operator abuse, or one-liners that obscure intent
- If you need a comment to explain a construct, rewrite the construct

## Descriptive Names

Names are the primary documentation. A good name makes the code explain itself.

- Variables, functions, classes, and modules must reveal intent at the call site
- Avoid single-letter names except in well-understood contexts (loop counters, math)
- Avoid abbreviations that are not universally understood in the domain
- Function names describe what they do: `calculateTax()` not `calc()`
- Boolean names read as assertions: `isEnabled`, `hasPermission`, `canRetry`

## Small, Single-Purpose Functions

A function does one thing at one level of abstraction.

- If a function needs a comment to explain what it does, split it
- Keep functions short enough to understand without scrolling
- Side effects are explicit: a function that reads a value does not also write one unless that is its stated purpose

## Avoid Deep Nesting: Use Early Returns

Each nesting level adds cognitive load. Flatten wherever possible.

- Guard clauses first: validate preconditions and return (or throw) early
- Avoid if/else chains in favour of early returns
- Target: no more than 2-3 levels of nesting in any function

**Example:**
```
// Nested
function process(order) {
  if (order != null) {
    if (order.isValid()) {
      if (order.hasItems()) {
        return ship(order)
      }
    }
  }
}

// Flat
function process(order) {
  if (order == null) return
  if (!order.isValid()) return
  if (!order.hasItems()) return
  return ship(order)
}
```

## Remove Duplication Through Meaningful Abstraction

When logic appears in two places, extract it: but only when the boundary is clear.

- Wait until you see the pattern a second time before abstracting
- The extracted name adds meaning; it does not just wrap the implementation
- A bad abstraction is worse than duplication: name it after *why* it exists, not *what* it does

## Domain-Centric Module Structure

Organise modules around business concepts, not technical layers.

- Avoid `/utils`, `/helpers`, `/common`
- Use `/order`, `/payment`, `/user`: things that change together, grouped together
- The folder structure communicates the domain, not the framework

## Comments Only When Code Cannot Express Intent

- Self-documenting code is the goal
- Write a comment only when the **why** cannot be expressed through naming or structure
- Never describe what the code does: that is the code's job
- Legal headers and public API documentation are acceptable exceptions

## Language Examples

When the project language is known, read the relevant reference for idiom-specific examples:

- `references/typescript.md`
- `references/python.md`
- `references/java.md`
- `references/scala.md`
- `references/javascript.md`

## Every Code Block Must Be Readable, Testable, and Changeable

Before committing any block, verify all three:
- **Readable:** Another developer understands it without explanation
- **Testable:** It can be verified in isolation without complex setup
- **Changeable:** Modifying one part does not force changes in unrelated parts
