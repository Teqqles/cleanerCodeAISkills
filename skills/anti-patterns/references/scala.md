# Anti-Patterns and Code Smells: Scala Examples

## God Class

```scala
// Smell: one class, many unrelated responsibilities
class OrderManager(db: Database, mailer: Mailer) {
  def placeOrder(cart: Cart): Order = ???
  def cancelOrder(id: String): Unit = ???
  def checkInventory(sku: String): Int = ???
  def reserveStock(sku: String, qty: Int): Unit = ???
  def sendConfirmation(order: Order): Unit = ???
  def generateReport(): Report = ???
}

// Fix: focused classes
class OrderService(db: Database) {
  def place(cart: Cart): Order = ???
  def cancel(id: String): Unit = ???
}

class InventoryService(db: Database) {
  def checkStock(sku: String): Int = ???
  def reserve(sku: String, qty: Int): Unit = ???
}

class OrderNotifications(mailer: Mailer) {
  def sendConfirmation(order: Order): Unit = ???
}
```

## Feature Envy

```scala
// Smell: reaches into customer internals
def formatInvoiceAddress(order: Order): String = {
  val c = order.customer
  val a = c.address
  s"${c.firstName} ${c.lastName}\n${a.street}\n${a.city}, ${a.postcode}"
}

// Fix: behaviour on the owning type
case class Customer(firstName: String, lastName: String, address: Address) {
  def formatAddress: String =
    s"$firstName $lastName\n${address.street}\n${address.city}, ${address.postcode}"
}
```

## Primitive Obsession

```scala
// Smell: bare strings with no validation boundary
def createUser(email: String, phone: String, role: String): User =
  if (!email.contains("@")) throw new IllegalArgumentException("invalid email")
  else User(email, phone, role)

// Fix: opaque types or value classes with validation
case class EmailAddress private (value: String)
object EmailAddress {
  def apply(raw: String): Either[String, EmailAddress] =
    if (raw.contains("@")) Right(new EmailAddress(raw.toLowerCase))
    else Left(s"Invalid email: $raw")
}

case class PhoneNumber private (value: String)
object PhoneNumber {
  def apply(raw: String): Either[String, PhoneNumber] = {
    val digits = raw.filter(_.isDigit)
    if (digits.length >= 10) Right(new PhoneNumber(digits))
    else Left(s"Invalid phone: $raw")
  }
}
```

## Flag Arguments

```scala
// Smell: boolean selects behaviour
def sendNotification(user: User, urgent: Boolean): Unit =
  if (urgent) sms.send(user.phone, buildUrgentMessage(user))
  else email.send(user.email, buildStandardMessage(user))

// Fix: named methods
def sendUrgentNotification(user: User): Unit =
  sms.send(user.phone, buildUrgentMessage(user))

def sendStandardNotification(user: User): Unit =
  email.send(user.email, buildStandardMessage(user))
```

## Temporal Coupling

```scala
// Smell: must call init() before process()
class ReportGenerator {
  def init(config: Config): Unit = ???
  def loadData(): Unit = ???
  def process(): Report = ???
}

// Fix: enforce through the API using a builder or single entry point
object ReportGenerator {
  def generate(config: Config): Report = {
    val data = loadData(config)
    processData(data)
  }
}
```

## Data Clumps

```scala
// Smell: lat/lon always travel together
def calculateDistance(lat1: Double, lon1: Double, lat2: Double, lon2: Double): Double = ???
def formatLocation(lat: Double, lon: Double): String = ???

// Fix: case class names the concept
case class Coordinate(latitude: Double, longitude: Double)

def calculateDistance(from: Coordinate, to: Coordinate): Double = ???
def formatLocation(coord: Coordinate): String = ???
```

## Speculative Generality with Type Classes

```scala
// Smell: type class with a single instance, added "for flexibility"
trait Serializable[A] {
  def serialize(a: A): String
}

// Only one instance ever exists
given Serializable[Order] with {
  def serialize(order: Order): String = order.toJson
}

// Fix: just call toJson directly until a second use case arrives
```
