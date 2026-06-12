# Dependency Injection: JavaScript Examples

JavaScript has no interface keyword. Define the expected shape through JSDoc or convention, and inject collaborators through the constructor.

## Inject Through the Constructor

```javascript
// domain/order/OrderService.js
class OrderService {
  constructor(orderRepository, clock) {
    this.orders = orderRepository
    this.clock = clock
  }

  async placeOrder(cart) {
    const order = Order.create(cart, this.clock.now())
    await this.orders.save(order)
    return order
  }
}
```

## In-Memory Fake for Tests

```javascript
// tests/fakes/InMemoryOrderRepository.js
class InMemoryOrderRepository {
  constructor() {
    this.store = new Map()
  }

  async save(order) {
    this.store.set(order.id, order)
  }

  async findById(id) {
    return this.store.get(id) ?? null
  }
}

class FakeClock {
  constructor(fixed) {
    this.fixed = fixed
  }

  now() {
    return this.fixed
  }
}
```

## Composition Root

```javascript
// main.js: only place where concrete types appear together
const orderService = new OrderService(
  new PostgresOrderRepository(db),
  new SystemClock()
)
```

## JSDoc as a Lightweight Contract

```javascript
/**
 * @typedef {Object} OrderRepository
 * @property {function(Order): Promise<void>} save
 * @property {function(string): Promise<Order|null>} findById
 */
```
