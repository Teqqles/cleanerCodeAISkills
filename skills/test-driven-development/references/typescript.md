# Test-Driven Development: TypeScript Examples

Test framework: **Jest** (`npm install --save-dev jest ts-jest @types/jest`)

## Red: Write the Failing Test First

```typescript
// PriceCalculator.test.ts
describe('PriceCalculator', () => {
  it('applies percentage discount to base price', () => {
    const calculator = new PriceCalculator()
    const result = calculator.applyDiscount(100, 0.1)
    expect(result).toBe(90)
  })
})
// Run: npx jest: test fails, PriceCalculator does not exist yet
```

## Green: Minimum Code to Pass

```typescript
// PriceCalculator.ts
export class PriceCalculator {
  applyDiscount(price: number, discountRate: number): number {
    return price - price * discountRate
  }
}
// Run: npx jest: test passes
```

## Refactor: Improve Without Breaking

```typescript
export class PriceCalculator {
  applyDiscount(price: number, discountRate: number): number {
    const discountAmount = this.discountAmountFor(price, discountRate)
    return price - discountAmount
  }

  private discountAmountFor(price: number, rate: number): number {
    return price * rate
  }
}
// Run: npx jest: still passes
```

## Test Names Describe Behaviour

```typescript
describe('PriceCalculator', () => {
  it('returns full price when discount rate is zero', () => {
    expect(new PriceCalculator().applyDiscount(100, 0)).toBe(100)
  })

  it('throws when discount rate exceeds 1', () => {
    expect(() => new PriceCalculator().applyDiscount(100, 1.5))
      .toThrow(InvalidDiscountError)
  })

  it('throws when price is negative', () => {
    expect(() => new PriceCalculator().applyDiscount(-1, 0.1))
      .toThrow(InvalidPriceError)
  })
})
```
