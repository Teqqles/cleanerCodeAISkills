# Dependency Injection: Scala Examples

## Define the Interface in the Domain Layer

```scala
// domain/order/OrderRepository.scala
trait OrderRepository {
  def save(order: Order): Unit
  def findById(id: String): Option[Order]
}

// domain/order/Clock.scala
trait Clock {
  def now(): Instant
}
```

## Inject Through the Constructor

```scala
// domain/order/OrderService.scala
class OrderService(orders: OrderRepository, clock: Clock) {
  def placeOrder(cart: Cart): Order = {
    val order = Order.create(cart, placedAt = clock.now())
    orders.save(order)
    order
  }
}
```

## In-Memory Fake for Tests

```scala
// tests/fakes/InMemoryOrderRepository.scala
class InMemoryOrderRepository extends OrderRepository {
  private var store = Map.empty[String, Order]

  def save(order: Order): Unit =
    store = store + (order.id -> order)

  def findById(id: String): Option[Order] =
    store.get(id)
}

class FakeClock(fixed: Instant) extends Clock {
  def now(): Instant = fixed
}
```

## Composition Root

```scala
// Main.scala: only place where concrete types appear together
object Main extends App {
  val orders = new PostgresOrderRepository(db)
  val clock  = new SystemClock
  val service = new OrderService(orders, clock)
}
```

For larger Scala applications, use constructor injection with a DI framework (ZIO, MacWire, Guice): but keep `OrderService` itself unaware of the container.
