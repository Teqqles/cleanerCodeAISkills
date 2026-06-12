# Quality Testing: TypeScript Examples

Test framework: **Jest**

## R: Right (correct output)

```typescript
it('returns order total from line items', () => {
  const order = new Order([new LineItem('book', 10), new LineItem('pen', 2)])
  expect(order.total()).toBe(12)
})
```

## I: Inverse (roundtrip)

```typescript
it('serialising then deserialising an order returns the original', () => {
  const order = Order.create({ customerId: 'c1', items: [new LineItem('book', 10)] })
  const restored = Order.fromJSON(order.toJSON())
  expect(restored).toEqual(order)
})
```

## H: Harmful (invalid inputs)

```typescript
it('throws when line item price is negative', () => {
  expect(() => new LineItem('book', -1)).toThrow(InvalidPriceError)
})

it('throws when item name is empty', () => {
  expect(() => new LineItem('', 10)).toThrow(InvalidItemError)
})
```

## T: Time (fake clock)

```typescript
it('marks order as expired after 30 minutes', () => {
  const clock = new FakeClock(new Date('2024-01-01T09:00:00Z'))
  const order = Order.create(cart, clock)
  clock.advanceBy({ minutes: 31 })
  expect(order.isExpired(clock.now())).toBe(true)
})
```

## B: Boundaries

```typescript
it('returns zero total for an order with no items', () => {
  expect(new Order([]).total()).toBe(0)
})

it('handles an order with exactly one item', () => {
  expect(new Order([new LineItem('book', 10)]).total()).toBe(10)
})
```

## E: Error Conditions

```typescript
it('throws OrderNotFoundError when order id does not exist', async () => {
  const repo = new InMemoryOrderRepository()
  await expect(repo.findById('missing')).rejects.toThrow(OrderNotFoundError)
})
```
