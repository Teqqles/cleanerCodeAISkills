# API Design: JavaScript Examples

## Explicit Inputs: No Hidden Dependencies

```javascript
// Bad: hidden dependency on global state
async function getRecommendations() {
  const user = currentSession.user          // hidden
  const region = process.env.REGION         // hidden
  return recommendationEngine.get(user, region)
}

// Good: all inputs declared
async function getRecommendations(userId, region, engine) {
  return engine.get(userId, region)
}
```

## Minimal Parameter Surface

```javascript
// Bad: seven positional parameters
function createOrder(customerId, skus, qtys, prices, addressLine1, addressLine2, city) { ... }

// Good: options object groups related inputs
function createOrder({ customerId, items, shippingAddress }) { ... }

// Caller is readable:
createOrder({
  customerId: 'c1',
  items: [{ sku: 'BOOK1', qty: 1, price: 10 }],
  shippingAddress: { line1: '1 High St', city: 'London' }
})
```

## Contract-First Documentation

```javascript
/**
 * Places a new order for the given customer.
 *
 * @param {CreateOrderRequest} request
 * @returns {Promise<Order>} The created order with a server-assigned id.
 * @throws {InvalidOrderError} if items is empty.
 * @throws {CustomerNotFoundError} if customerId does not exist.
 */
async function placeOrder(request) { ... }
```

## Stable Versioning

```javascript
// Add optional fields: backwards compatible
function createOrder({
  customerId,
  items,
  shippingAddress,
  promoCode = null       // added later: optional, safe
}) { ... }
```
