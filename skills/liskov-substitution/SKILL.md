---
name: liskov-substitution
description: Applies the Liskov Substitution Principle when designing type hierarchies, interfaces, and polymorphic code. Use this skill whenever you are creating subtypes, implementing interfaces, overriding methods, or reviewing inheritance hierarchies. Trigger when the user says "should this extend that", "is this a valid subtype", "this override behaves differently", or when you notice subtypes that throw unexpected exceptions, ignore base class contracts, or require callers to check the concrete type before using them.
---

# Liskov Substitution Principle

You apply the Liskov Substitution Principle: any instance of a type must be replaceable by an instance of its subtype without altering the correctness of the program. If calling code must check the concrete type to behave correctly, the hierarchy is broken.

## The Rule

Code that works with a base type must continue to work correctly with any subtype, without knowing which subtype it received. The subtype must honour the contract of the base type in full.

- A subtype may do more (handle extra cases) but never less (reject cases the base type accepts)
- A subtype may return a narrower result but never a wider one
- A subtype may require less from callers but never more
- Callers must never need to inspect the concrete type to decide what to do

## Contracts, Not Just Signatures

LSP is about behavioural compatibility, not just matching method signatures.

A contract includes:
- **Preconditions:** what the method requires from callers. A subtype must not strengthen preconditions (must not reject inputs the base type accepts).
- **Postconditions:** what the method guarantees to callers. A subtype must not weaken postconditions (must not return results the base type would not).
- **Invariants:** properties that are always true. A subtype must preserve every invariant of the base type.

## Classic Violation: Square/Rectangle

```
class Rectangle {
  setWidth(w) { this.width = w }
  setHeight(h) { this.height = h }
  area() { return this.width * this.height }
}

class Square extends Rectangle {
  setWidth(w) { this.width = w; this.height = w }
  setHeight(h) { this.width = h; this.height = h }
}
```

A caller that sets width and height independently expects `area() == width * height`. Square breaks this. The fix is not to make Square extend Rectangle. They share geometry but not behaviour.

## Symptoms of Violated LSP

- A subtype throws `NotImplementedError` or `UnsupportedOperationError` for inherited methods
- Calling code uses `instanceof` / `is` / type checks to decide behaviour
- A subtype ignores or silently discards parameters the base type uses
- Tests for the base type fail when run against the subtype
- Documentation says "do not call X on this subtype"

## The instanceof Test

If you find code like this, LSP is violated somewhere:

```
function process(shape: Shape) {
  if (shape instanceof Circle) {
    // special handling
  } else if (shape instanceof Square) {
    // different handling
  }
}
```

The caller should not need to know the concrete type. If it does, either the interface is wrong or the subtype does not honour the contract.

## Rules for Correct Subtypes

1. **Accept at least what the base accepts.** Never add preconditions. If the base accepts null, the subtype accepts null.
2. **Deliver at least what the base promises.** Never weaken postconditions. If the base guarantees a non-null return, the subtype returns non-null.
3. **Preserve invariants.** If the base says balance is never negative, the subtype maintains that.
4. **Throw only exceptions the base permits.** New exception types that callers do not expect violate the contract.
5. **Do not remove or ignore behaviour.** A no-op override that silently swallows a call is a lie to the caller.

## When Not to Use Inheritance

LSP violations often signal that inheritance was the wrong tool:

- "Is-a" must mean "behaves-as", not "shares-fields-with"
- If the subtype cannot honour the full contract, use composition instead
- If you find yourself overriding to restrict, the hierarchy is backwards
- Prefer interfaces/traits that define capability over abstract base classes that assume structure

**Example:**

```
// Violation: ReadOnlyList extends List but throws on add()
class ReadOnlyList extends List {
  add(item) { throw new UnsupportedOperationError() }
}

// Fix: separate interfaces
interface Readable {
  get(index): Item
  size(): number
}

interface Writable extends Readable {
  add(item): void
}
```

## Testing LSP

Write contract tests against the base type interface, then run them against every implementation:

- If all implementations pass the same contract tests, LSP holds
- If any implementation needs special-case tests that skip base type assertions, the substitution is broken
- Contract tests are the mechanical proof of LSP compliance

## Language Examples

When the project language is known, read the relevant reference for idiom-specific examples:

- `references/typescript.md`
- `references/python.md`
- `references/java.md`
- `references/scala.md`
- `references/javascript.md`

## LSP Enables Polymorphism

The entire value of polymorphism depends on LSP. Without it, every call site must know which concrete type it holds, and the abstraction is a lie. Honour the contract, or do not claim the type.
