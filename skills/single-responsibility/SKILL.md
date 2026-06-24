---
name: single-responsibility
description: Applies the Single Responsibility Principle when designing or reviewing classes, modules, and functions. Use this skill whenever you are deciding what belongs in a class, where to split a module, or how to scope a function. Trigger when the user says "this class is doing too much", "where does this belong", "should I split this", or when you notice a class that changes for multiple unrelated reasons, a module with imports from unrelated domains, or a function that mixes orchestration with detail work.
---

# Single Responsibility Principle

You apply the Single Responsibility Principle: a class, module, or function should have one reason to change. "One reason to change" means one stakeholder or one axis of business change that would require editing this code.

## What "One Reason to Change" Means

It does not mean "does one thing" in the mechanical sense. It means: only one category of business change forces a modification.

- A class that formats reports AND queries data changes when the report layout changes AND when the data schema changes. Two reasons. Split it.
- A class that validates orders and enforces business rules around order validity has one reason to change: the rules of what makes an order valid. Keep it together.
- The test: name the stakeholder who would ask for the change. If two different stakeholders could independently request changes to the same class, it has too many responsibilities.

## Identify Responsibilities by Asking "Who Requests This Change?"

- If the answer is "the finance team" for one method and "the operations team" for another, those are separate responsibilities
- Group code by the reason it changes, not by the data it touches
- Two methods that touch the same field but serve different stakeholders belong in different classes

## Symptoms of Violated SRP

- A class has methods that cluster into groups that never call each other
- A change to one feature forces unrelated tests to break
- The class name includes "And" or "Manager" or "Handler" with no further qualification
- Imports span unrelated domains (e.g., a class importing both payment gateway and email libraries)
- You struggle to name the class without using a generic word

## How to Split

When a class has multiple responsibilities:

1. Identify the distinct axes of change
2. Extract each into its own class named after what it does
3. The original class becomes either a thin coordinator or disappears entirely
4. Each new class is independently testable

**Example:**

```
// Two responsibilities: persistence logic and notification logic
class UserService {
  save(user) { /* writes to database */ }
  sendWelcomeEmail(user) { /* formats and sends email */ }
  notifyAdmin(user) { /* sends Slack message */ }
}

// Split: each class changes for one reason
class UserRepository {
  save(user) { /* writes to database */ }
}

class UserNotifications {
  sendWelcome(user) { /* formats and sends email */ }
  notifyAdmin(user) { /* sends Slack message */ }
}
```

## SRP Applies at Every Scale

- **Function level:** a function operates at one level of abstraction. It either orchestrates or does detail work, not both.
- **Class level:** a class groups operations that change together for the same reason.
- **Module level:** a module encapsulates a single domain concept. Other modules depend on its interface, not its internals.

## Cohesion Is the Positive Framing

High cohesion means everything in the unit is closely related and changes together. SRP is the constraint; cohesion is the quality you get when you follow it.

- If removing a method from a class would make the class incomplete or broken, the method belongs there
- If removing a method would leave the class perfectly coherent, the method belongs elsewhere

## Do Not Over-Split

SRP does not mean one method per class. It means one reason to change per class.

- A `DateRange` class with `contains()`, `overlaps()`, and `duration()` has one responsibility: representing and querying a date range
- Splitting each method into its own class would scatter a cohesive concept
- The test: would any of these methods ever change independently for different business reasons? If not, keep them together.

## Language Examples

When the project language is known, read the relevant reference for idiom-specific examples:

- `references/typescript.md`
- `references/python.md`
- `references/java.md`
- `references/scala.md`
- `references/javascript.md`

## SRP Enables Change

The goal is not purity for its own sake. The goal is that when one business requirement changes, you edit one class, run its tests, and deploy. Nothing else is affected.
