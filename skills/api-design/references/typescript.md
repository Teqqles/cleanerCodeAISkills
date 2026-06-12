# API Design: TypeScript Examples

## Explicit Inputs: No Hidden Dependencies

```typescript
// Bad: hidden dependency on global state
async function getRecommendations(): Promise<Product[]> {
  const user = currentSession.user          // hidden
  const region = process.env.REGION         // hidden
  return recommendationEngine.get(user, region)
}

// Good: all inputs declared
async function getRecommendations(
  userId: string,
  region: string,
  engine: RecommendationEngine
): Promise<Product[]> {
  return engine.get(userId, region)
}
```

## Minimal Parameter Surface

```typescript
// Bad: caller must know internal field names
function createOrder(
  customerId: string,
  lineItemSkus: string[],
  lineItemQtys: number[],
  lineItemPrices: number[],
  shippingAddressLine1: string,
  shippingAddressLine2: string,
  shippingCity: string
): Order { ... }

// Good: group related inputs into a value object
interface CreateOrderRequest {
  customerId: string
  items: Array<{ sku: string; qty: number; price: number }>
  shippingAddress: Address
}

function createOrder(request: CreateOrderRequest): Order { ... }
```

## Contract-First Documentation

```typescript
/**
 * Places a new order for the given customer.
 *
 * @throws InvalidOrderError if items is empty.
 * @throws CustomerNotFoundError if customerId does not exist.
 * @returns The created Order with a server-assigned id.
 */
async function placeOrder(request: CreateOrderRequest): Promise<Order> { ... }
```

## Stable Versioning

```typescript
// Add optional fields: backwards compatible
interface CreateOrderRequest {
  customerId: string
  items: Array<{ sku: string; qty: number; price: number }>
  shippingAddress: Address
  promoCode?: string      // added in v2: optional, safe
}

// Breaking change: new version required
// v1: placeOrder(request: CreateOrderRequestV1)
// v2: placeOrder(request: CreateOrderRequestV2)
```
