# Engineering Rigor: Scala Examples

## Typed Errors: No Silent Failures

```scala
// Bad: exception swallowed
def chargeCustomer(order: Order): Unit =
  try paymentGateway.charge(order)
  catch { case _: Exception => () }  // silently ignored

// Good: typed result using Either
sealed trait ChargeResult
case class ChargeSuccess(chargeId: String) extends ChargeResult
case class ChargeFailure(reason: String)   extends ChargeResult

def chargeCustomer(order: Order): ChargeResult =
  try {
    val chargeId = paymentGateway.charge(order)
    ChargeSuccess(chargeId)
  } catch {
    case e: PaymentGatewayException =>
      logger.error("charge_failed orderId={} error={}", order.id, e.getMessage)
      ChargeFailure("gateway_error")
  }
```

## Explicit Precondition Enforcement

```scala
// Bad: wrong result with no explanation
def applyDiscount(price: BigDecimal, rate: BigDecimal): BigDecimal =
  price - price * rate

// Good: preconditions checked at the boundary
def applyDiscount(price: BigDecimal, rate: BigDecimal): BigDecimal = {
  require(price >= 0, s"price must be non-negative, got $price")
  require(rate >= 0 && rate <= 1, s"rate must be 0-1, got $rate")
  price - price * rate
}
```

## Structured Logging

```scala
// Bad: unstructured
println(s"order placed for user $userId")

// Good: structured (SLF4J placeholder style)
logger.info("order_placed orderId={} customerId={} totalAmount={} itemCount={}",
  order.id, order.customerId, order.total.toString, order.items.size)
```

## Environment-Driven Configuration

```scala
// Bad: hardcoded
val DbUrl = "postgres://localhost:5432/mydb"

// Good: from environment
val dbUrl = sys.env.getOrElse("DATABASE_URL",
  throw new ConfigurationException("DATABASE_URL is required"))
```
