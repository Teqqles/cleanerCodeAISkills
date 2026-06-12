# Code Style: Scala

## Formatter

```bash
sbt scalafmt
```

Typical config: `.scalafmt.conf`

## Naming Conventions (Scala Style Guide)

```scala
val userName = "Alice"                  // camelCase: vals and vars
def calculateTotal(): BigDecimal = {}   // camelCase: methods
class OrderService {}                   // PascalCase: classes
trait UserRepository {}                 // PascalCase: traits
case class UserId(value: String)        // PascalCase: case classes
object PaymentMethod {}                 // PascalCase: objects
val MaxRetries = 3                      // PascalCase: top-level immutable vals (Scala convention)
```

## Comments

```scala
// Bad: describes what, not why
// Sum all user balances
val total = users.map(_.balance).sum

// Good: only when why is non-obvious
// Balances can be negative (credit accounts); sum without filtering is intentional
val total = users.map(_.balance).sum
```

## Scaladoc: Contracts, Not Implementation

```scala
// Bad
/** This method loops through orders and returns the total */
def calculateTotal(orders: List[Order]): BigDecimal = ???

// Good
/**
 * Returns the sum of all order amounts.
 * Returns zero for an empty list.
 */
def calculateTotal(orders: List[Order]): BigDecimal = ???
```
