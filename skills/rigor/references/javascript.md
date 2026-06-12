# Engineering Rigor: JavaScript Examples

## Typed Errors: No Silent Failures

```javascript
// Bad: error swallowed
async function chargeCustomer(order) {
  try {
    await paymentGateway.charge(order)
  } catch {
    // silently ignored
  }
}

// Good: result object, failure surfaced
async function chargeCustomer(order) {
  try {
    const chargeId = await paymentGateway.charge(order)
    return { success: true, chargeId }
  } catch (err) {
    logger.error('charge_failed', { orderId: order.id, err: err.message })
    return { success: false, reason: 'gateway_error' }
  }
}
```

## Explicit Precondition Enforcement

```javascript
// Bad: crashes with unhelpful message later
function applyDiscount(price, rate) {
  return price - price * rate
}

// Good: preconditions checked at the boundary
function applyDiscount(price, rate) {
  if (price < 0) throw new InvalidPriceError(`price must be non-negative, got ${price}`)
  if (rate < 0 || rate > 1) throw new InvalidDiscountError(`rate must be 0-1, got ${rate}`)
  return price - price * rate
}
```

## Structured Logging

```javascript
// Bad: unstructured
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

```javascript
// Bad: hardcoded
const DB_URL = 'postgres://localhost:5432/mydb'

// Good: from environment
const DB_URL = process.env.DATABASE_URL
if (!DB_URL) throw new ConfigurationError('DATABASE_URL is required')
```
