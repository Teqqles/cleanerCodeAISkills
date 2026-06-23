---
name: anti-patterns
description: Identifies and avoids code smells and anti-patterns that degrade maintainability, readability, and extensibility. Use this skill whenever you are writing or reviewing code. Trigger when the user says "this feels wrong", "is this a smell", "what's wrong with this code", "review this for quality", or when you notice classes doing too much, methods with too many parameters, duplicated structures, inappropriate intimacy between modules, or code that resists change. Apply these checks by default when writing any code.
---

# Anti-Patterns and Code Smells

You recognise and avoid code smells and anti-patterns. These are the shared vocabulary for describing what is wrong with code and why it resists change.

## Code Smells Are Signals, Not Sins

A smell indicates a likely structural problem. It does not always require action, but it always requires thought.

- A smell is a heuristic: "this often causes pain"
- Multiple smells in the same area compound: fix them together
- A smell in isolation might be acceptable; a cluster is always a problem
- Name the smell when you spot it: shared vocabulary accelerates design conversations

## God Class / Blob

A class that knows too much and does too much. It attracts responsibility because it already has context.

- Symptom: the class has dozens of methods across unrelated concerns
- Symptom: most changes to the system require editing this class
- Fix: extract cohesive groups of methods into their own classes
- A class with more than one reason to change violates the single responsibility principle

## Feature Envy

A method that uses more data from another class than from its own.

- Symptom: chains of getters reaching into another object's internals
- Symptom: the method would make more sense living on the other class
- Fix: move the method to the class whose data it actually uses
- This smell often signals misplaced responsibility

## Shotgun Surgery

A single logical change requires editing many classes in many places.

- Symptom: adding a field means touching 5+ files
- Symptom: related logic is scattered rather than grouped
- Fix: consolidate the scattered logic into a single module or class
- The opposite of God Class, but equally painful

## Primitive Obsession

Using primitive types (strings, ints, booleans) where a domain type would communicate intent.

- Symptom: `string email`, `string phoneNumber`, `int status` with no validation boundary
- Symptom: the same validation logic repeated everywhere the primitive is used
- Fix: wrap in a value type that validates at construction: `EmailAddress`, `OrderStatus`
- Domain types make invalid states unrepresentable

## Long Parameter Lists

A function that takes many arguments signals it is doing too much or its dependencies are poorly structured.

- Symptom: 5+ parameters, especially of the same type
- Symptom: callers constantly pass `null` or default values for parameters they do not use
- Fix: group related parameters into a value object
- Fix: split the function into smaller ones that each need fewer arguments

## Flag Arguments

A boolean parameter that selects between two behaviours inside one function.

- Symptom: `process(order, true)` where `true` means "rush"
- Symptom: the function body is an if/else split on the flag
- Fix: two separate functions with descriptive names: `processStandard()`, `processRush()`
- A flag argument hides two functions pretending to be one

## Inappropriate Intimacy

Two classes that know too much about each other's internals.

- Symptom: class A reaches into class B's private fields or internal structure
- Symptom: changing class B's internals breaks class A
- Fix: define a public interface on B that exposes only what A needs
- Fix: introduce an intermediary if the coupling cannot be reduced directly

## Speculative Generality

Code written to handle cases that do not exist yet and may never exist.

- Symptom: abstract classes with a single implementation
- Symptom: parameters, hooks, or extension points that nothing uses
- Symptom: "we might need this later"
- Fix: delete it. Write it when you need it. YAGNI.
- Unused abstractions confuse every reader who tries to understand why they exist

## Temporal Coupling

Code that must be called in a specific sequence but does not enforce or communicate that sequence.

- Symptom: calling methods out of order causes subtle bugs
- Symptom: `init()` must be called before `process()` but nothing prevents the reverse
- Fix: make the sequence impossible to violate through the type system or API design
- Fix: combine the steps into a single operation that guarantees the order

## Data Clumps

Groups of data that always appear together but are not captured in a named structure.

- Symptom: the same 3-4 parameters travel together across many function signatures
- Symptom: parallel arrays or maps that represent one conceptual entity
- Fix: extract a class or record that names the concept

## Language Examples

When the project language is known, read the relevant reference for smell identification and fix patterns:

- `references/typescript.md`
- `references/python.md`
- `references/java.md`
- `references/scala.md`
- `references/javascript.md`

## The Compound Effect

Individual smells are warnings. Clusters are alarms. When you find one smell, look for its neighbours: God Classes attract Feature Envy; Primitive Obsession breeds Data Clumps; Shotgun Surgery follows from Inappropriate Intimacy. Fix the root, not the leaf.
