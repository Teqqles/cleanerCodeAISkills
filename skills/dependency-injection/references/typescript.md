# Dependency Injection: TypeScript Examples

## Define the Interface in the Domain Layer

```typescript
// domain/order/OrderRepository.ts
export interface OrderRepository {
  save(order: Order): Promise<void>
  findById(id: string): Promise<Order | null>
}

export interface Clock {
  now(): Date
}
```

## Inject Through the Constructor

```typescript
// domain/order/OrderService.ts
export class OrderService {
  constructor(
    private readonly orders: OrderRepository,
    private readonly clock: Clock
  ) {}

  async placeOrder(cart: Cart): Promise<Order> {
    const order = Order.create(cart, this.clock.now())
    await this.orders.save(order)
    return order
  }
}
```

## In-Memory Fake for Tests

```typescript
// tests/fakes/InMemoryOrderRepository.ts
export class InMemoryOrderRepository implements OrderRepository {
  private readonly store = new Map<string, Order>()

  async save(order: Order): Promise<void> {
    this.store.set(order.id, order)
  }

  async findById(id: string): Promise<Order | null> {
    return this.store.get(id) ?? null
  }
}

export class FakeClock implements Clock {
  constructor(private readonly fixed: Date) {}
  now(): Date { return this.fixed }
}
```

## Composition Root

```typescript
// main.ts: only place where concrete types appear together
const orderService = new OrderService(
  new PostgresOrderRepository(db),
  new SystemClock()
)
```
