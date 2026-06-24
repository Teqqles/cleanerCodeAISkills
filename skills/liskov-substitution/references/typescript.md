# Liskov Substitution: TypeScript Examples

## Violation: Subtype That Throws on Inherited Methods

```typescript
interface Collection<T> {
  add(item: T): void
  get(index: number): T
  size(): number
}

class ReadOnlyList<T> implements Collection<T> {
  add(item: T): void {
    throw new Error('Cannot add to read-only list')  // violates LSP
  }
  get(index: number): T { /* ... */ }
  size(): number { /* ... */ }
}
```

A caller using `Collection<T>` expects `add()` to work. Fix: separate interfaces.

```typescript
interface Readable<T> {
  get(index: number): T
  size(): number
}

interface Writable<T> extends Readable<T> {
  add(item: T): void
}

class ReadOnlyList<T> implements Readable<T> {
  get(index: number): T { /* ... */ }
  size(): number { /* ... */ }
}
```

## Violation: Subtype That Strengthens Preconditions

```typescript
interface Discount {
  apply(amount: number): number
}

class PercentDiscount implements Discount {
  apply(amount: number): number {
    return amount * 0.9
  }
}

class MinimumOrderDiscount implements Discount {
  apply(amount: number): number {
    if (amount < 50) throw new Error('Order too small')  // strengthened precondition
    return amount * 0.8
  }
}
```

Callers of `Discount` do not expect `apply()` to reject amounts. Fix: handle the condition without throwing.

```typescript
class MinimumOrderDiscount implements Discount {
  apply(amount: number): number {
    if (amount < 50) return amount  // no discount, no exception
    return amount * 0.8
  }
}
```

## Contract Tests Proving LSP

```typescript
function discountContractTests(createDiscount: () => Discount) {
  const discount = createDiscount()

  test('applies to any non-negative amount', () => {
    expect(discount.apply(0)).toBeGreaterThanOrEqual(0)
    expect(discount.apply(10)).toBeGreaterThanOrEqual(0)
    expect(discount.apply(1000)).toBeGreaterThanOrEqual(0)
  })

  test('never returns more than the input', () => {
    expect(discount.apply(100)).toBeLessThanOrEqual(100)
  })
}

// Run against every implementation
discountContractTests(() => new PercentDiscount())
discountContractTests(() => new MinimumOrderDiscount())
discountContractTests(() => new BulkDiscount())
```

## Violation: Type Checks in Calling Code

```typescript
// Caller forced to inspect concrete type
function render(shape: Shape): string {
  if (shape instanceof Circle) {
    return `circle(${shape.radius})`
  } else if (shape instanceof Rectangle) {
    return `rect(${shape.width}, ${shape.height})`
  }
  throw new Error('Unknown shape')
}

// Fixed: polymorphism through the interface
interface Shape {
  render(): string
}

class Circle implements Shape {
  constructor(private radius: number) {}
  render(): string { return `circle(${this.radius})` }
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  render(): string { return `rect(${this.width}, ${this.height})` }
}

function render(shape: Shape): string {
  return shape.render()
}
```
