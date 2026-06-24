# Single Responsibility: Scala Examples

## Splitting a Multi-Responsibility Class

```scala
// Violated: UserService handles persistence, validation, and notifications
class UserService(db: Database, mailer: Mailer) {
  def register(data: UserInput): User = {
    if (!data.email.contains("@")) throw new InvalidInputException("bad email")
    val user = User(data.email, data.name)
    db.save(user)
    mailer.send(user.email, "Welcome!")
    user
  }
}

// Fixed: each class has one reason to change
class UserValidator {
  def validate(data: UserInput): Unit =
    if (!data.email.contains("@")) throw new InvalidInputException("bad email")
}

class UserRepository(db: Database) {
  def save(user: User): Unit = db.save(user)
}

class WelcomeNotifier(mailer: Mailer) {
  def notify(user: User): Unit = mailer.send(user.email, "Welcome!")
}

class RegisterUser(validator: UserValidator, repo: UserRepository, notifier: WelcomeNotifier) {
  def execute(data: UserInput): User = {
    validator.validate(data)
    val user = User(data.email, data.name)
    repo.save(user)
    notifier.notify(user)
    user
  }
}
```

## Function-Level SRP: Separate Orchestration from Detail

```scala
// Violated: mixes calculation with formatting
def generateReport(orders: List[Order]): String = {
  val total = orders.map(_.amount).sum
  val lines = orders.map(o => f"${o.id}: ${o.amount}%.2f")
  s"Report\n${lines.mkString("\n")}\nTotal: ${total}%.2f"
}

// Fixed: each function does one thing
def calculateTotal(orders: List[Order]): BigDecimal =
  orders.map(_.amount).sum

def formatReport(orders: List[Order], total: BigDecimal): String = {
  val lines = orders.map(o => f"${o.id}: ${o.amount}%.2f")
  s"Report\n${lines.mkString("\n")}\nTotal: ${total}%.2f"
}

def generateReport(orders: List[Order]): String = {
  val total = calculateTotal(orders)
  formatReport(orders, total)
}
```

## Package-Level SRP

```
// Violated: order package mixes concerns
com.example.order/
├── Order.scala
├── PlaceOrder.scala
├── ChargeCard.scala       // payment concern
└── RefundPayment.scala    // payment concern

// Fixed: separate packages by axis of change
com.example.order/
├── Order.scala
└── PlaceOrder.scala

com.example.payment/
├── ChargeCard.scala
└── RefundPayment.scala
```

## Traits for Composing Single-Responsibility Units

```scala
// Each trait represents one cohesive responsibility
trait OrderValidation {
  def validate(order: Order): Either[String, Order]
}

trait OrderPersistence {
  def save(order: Order): Unit
  def find(id: String): Option[Order]
}

trait OrderNotification {
  def notifyPlaced(order: Order): Unit
}

// Composed at the boundary
class OrderWorkflow(
  validation: OrderValidation,
  persistence: OrderPersistence,
  notification: OrderNotification
) {
  def place(order: Order): Either[String, Order] =
    validation.validate(order).map { valid =>
      persistence.save(valid)
      notification.notifyPlaced(valid)
      valid
    }
}
```
