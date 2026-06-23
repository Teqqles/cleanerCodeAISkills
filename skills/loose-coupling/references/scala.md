# Loose Coupling: Scala Examples

## Depend on Abstractions

```scala
// Coupled: imports a concrete class
import infrastructure.PostgresUserStore

class UserService {
  private val store = new PostgresUserStore()
}

// Decoupled: depends on a trait the consumer defines
trait UserStore {
  def find(id: String): Option[User]
  def save(user: User): Unit
}

class UserService(store: UserStore) {
  def register(user: User): Unit = store.save(user)
}
```

## Law of Demeter

```scala
// Coupled: traverses the object graph
def shippingCity(order: Order): String =
  order.customer.address.city

// Decoupled: ask the object
def shippingCity(order: Order): String =
  order.shippingCity
```

## Tell, Don't Ask

```scala
// Ask: caller decides based on state
def maybeExpire(session: Session): Unit =
  if (System.currentTimeMillis() - session.lastActivity > session.timeout) {
    session.expired = true
    notifyUser(session.userId)
  }

// Tell: session owns the decision
class Session(userId: String, timeout: Long) {
  def checkExpiry(onExpired: String => Unit): Unit =
    if (isExpired) onExpired(userId)
}
```

## Interface Segregation

```scala
// Broad: one trait with everything
trait DataStore {
  def read(key: String): String
  def write(key: String, value: String): Unit
  def delete(key: String): Unit
  def listKeys: List[String]
  def watch(key: String)(cb: String => Unit): Unit
}

// Segregated: focused traits
trait Readable {
  def read(key: String): String
}

trait Writable {
  def write(key: String, value: String): Unit
}

trait Watchable {
  def watch(key: String)(cb: String => Unit): Unit
}
```

## Events Over Direct Calls

```scala
// Coupled: OrderService knows all consumers
class OrderService(
  inventory: InventoryService,
  email: EmailService,
  analytics: AnalyticsService
) {
  def place(cart: Cart): Order = {
    val order = Order.create(cart)
    inventory.reserve(order.items)
    email.sendConfirmation(order)
    analytics.track("order_placed", order)
    order
  }
}

// Decoupled: emit events
class OrderService {
  val orderPlaced = new EventEmitter[Order]

  def place(cart: Cart): Order = {
    val order = Order.create(cart)
    orderPlaced.emit(order)
    order
  }
}

// Composition root
orderService.orderPlaced.subscribe(order => inventory.reserve(order.items))
orderService.orderPlaced.subscribe(order => email.sendConfirmation(order))
```

## Extension Without Modification

```scala
// Coupled: match grows with every format
def exportReport(report: Report, format: String): String = format match {
  case "csv"  => toCsv(report)
  case "json" => toJson(report)
  case "xml"  => toXml(report)
  case other  => throw new IllegalArgumentException(s"Unknown: $other")
}

// Decoupled: new formats are new implementations
trait ReportExporter {
  def export(report: Report): String
}

class CsvExporter extends ReportExporter {
  def export(report: Report): String = toCsv(report)
}

class JsonExporter extends ReportExporter {
  def export(report: Report): String = toJson(report)
}
```

## Function Types as Lightweight Decoupling

Scala idiom: when the interface is a single method, a function type suffices:

```scala
// No trait needed for a single-method dependency
class NotificationService(send: (String, String) => Unit) {
  def notifyUser(userId: String, message: String): Unit =
    send(userId, message)
}

// Wire with any implementation
val service = new NotificationService(emailSender.send)
val testService = new NotificationService((_, _) => ())
```
