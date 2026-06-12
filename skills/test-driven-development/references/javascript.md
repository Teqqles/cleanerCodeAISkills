# Test-Driven Development: JavaScript Examples

Test framework: **Jest** (`npm install --save-dev jest`)

## Red: Write the Failing Test First

```javascript
// PriceCalculator.test.js
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

```javascript
// PriceCalculator.js
class PriceCalculator {
  applyDiscount(price, discountRate) {
    return price - price * discountRate
  }
}

module.exports = { PriceCalculator }
// Run: npx jest: test passes
```

## Refactor: Improve Without Breaking

```javascript
class PriceCalculator {
  applyDiscount(price, discountRate) {
    const discountAmount = this.#discountAmountFor(price, discountRate)
    return price - discountAmount
  }

  #discountAmountFor(price, rate) {
    return price * rate
  }
}
// Run: npx jest: still passes
```

## Test Names Describe Behaviour

```javascript
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
```
