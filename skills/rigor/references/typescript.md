# Engineering Rigor: TypeScript Examples

## Typed Errors: No Silent Failures

```typescript
// Bad: error swallowed
async function chargeCustomer(order: Order): Promise<void> {
  try {
    await paymentGateway.charge(order)
  } catch {
    // silently ignored
  }
}

// Good: typed result, failure surfaced
type ChargeResult =
  | { success: true;  chargeId: string }
  | { success: false; reason: 'declined' | 'gateway_error' }

async function chargeCustomer(order: Order): Promise<ChargeResult> {
  try {
    const chargeId = await paymentGateway.charge(order)
    return { success: true, chargeId }
  } catch (err) {
    logger.error('charge failed', { orderId: order.id, err })
    return { success: false, reason: 'gateway_error' }
  }
}
```

## Explicit Precondition Enforcement

```typescript
// Bad: crashes with unhelpful message later
function applyDiscount(price: number, rate: number): number {
  return price - price * rate
}

// Good: preconditions checked at the boundary
function applyDiscount(price: number, rate: number): number {
  if (price < 0) throw new InvalidPriceError(`price must be non-negative, got ${price}`)
  if (rate < 0 || rate > 1) throw new InvalidDiscountError(`rate must be 0-1, got ${rate}`)
  return price - price * rate
}
```

## Structured Logging

```typescript
// Bad: unstructured, hard to query
console.log('order placed for user ' + userId)

// Good: structured, filterable
logger.info('order_placed', {
  orderId: order.id,
  customerId: order.customerId,
  totalAmount: order.total(),
  itemCount: order.items.length
})
```

## Environment-Driven Configuration

```typescript
// Bad: hardcoded
const DB_URL = 'postgres://localhost:5432/mydb'

// Good: from environment
const DB_URL = process.env.DATABASE_URL
if (!DB_URL) throw new ConfigurationError('DATABASE_URL is required')
```
