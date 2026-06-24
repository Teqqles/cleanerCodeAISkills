# Liskov Substitution: JavaScript Examples

## Violation: Subtype That Throws on Inherited Methods

```javascript
class Collection {
  add(item) { /* ... */ }
  get(index) { /* ... */ }
  size() { /* ... */ }
}

class ReadOnlyList extends Collection {
  add(item) {
    throw new Error('Cannot add to read-only list')  // violates LSP
  }
}
```

Fix: do not extend a type whose contract you cannot honour. Use separate interfaces via duck typing.

```javascript
class ReadOnlyList {
  get(index) { /* ... */ }
  size() { /* ... */ }
}

// Callers that only need reading accept any object with get() and size()
function displayItems(readable) {
  for (let i = 0; i < readable.size(); i++) {
    console.log(readable.get(i))
  }
}
```

## Violation: Subtype That Strengthens Preconditions

```javascript
/** @typedef {{ apply(amount: number): number }} Discount */

class PercentDiscount {
  apply(amount) { return amount * 0.9 }
}

class MinimumOrderDiscount {
  apply(amount) {
    if (amount < 50) throw new Error('Order too small')  // strengthened precondition
    return amount * 0.8
  }
}
```

Fix: accept all inputs the contract permits.

```javascript
class MinimumOrderDiscount {
  apply(amount) {
    if (amount < 50) return amount  // no discount, no exception
    return amount * 0.8
  }
}
```

## Contract Tests Proving LSP

```javascript
function discountContractTests(createDiscount) {
  const discount = createDiscount()

  test('applies to any non-negative amount', () => {
    expect(discount.apply(0)).toBeGreaterThanOrEqual(0)
    expect(discount.apply(10)).toBeGreaterThanOrEqual(0)
    expect(discount.apply(1000)).toBeGreaterThanOrEqual(0)
  })

  test('never returns more than input', () => {
    expect(discount.apply(100)).toBeLessThanOrEqual(100)
  })
}

// Run against every implementation
discountContractTests(() => new PercentDiscount())
discountContractTests(() => new MinimumOrderDiscount())
discountContractTests(() => new BulkDiscount())
```

## Violation: Type Checks in Calling Code

```javascript
// Caller forced to check concrete type
function render(shape) {
  if (shape instanceof Circle) {
    return `circle(${shape.radius})`
  } else if (shape instanceof Rectangle) {
    return `rect(${shape.width}, ${shape.height})`
  }
  throw new Error('Unknown shape')
}

// Fixed: each shape knows how to render itself
class Circle {
  constructor(radius) { this.radius = radius }
  render() { return `circle(${this.radius})` }
}

class Rectangle {
  constructor(width, height) {
    this.width = width
    this.height = height
  }
  render() { return `rect(${this.width}, ${this.height})` }
}

function render(shape) {
  return shape.render()
}
```
