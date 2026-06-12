# API Design: Scala Examples

## Explicit Inputs: No Hidden Dependencies

```scala
// Bad: hidden dependency on implicit context
def getRecommendations()(implicit ctx: RequestContext): List[Product] = {
  val user   = ctx.currentUser
  val region = sys.env("REGION")
  recommendationEngine.get(user, region)
}

// Good: all inputs explicit
def getRecommendations(
  userId: String,
  region: String,
  engine: RecommendationEngine
): List[Product] =
  engine.get(userId, region)
```

## Minimal Parameter Surface

```scala
// Bad: caller must track parallel lists
def createOrder(
  customerId: String,
  skus: List[String],
  qtys: List[Int],
  prices: List[BigDecimal],
  addressLine1: String,
  addressLine2: String,
  city: String
): Order = ???

// Good: group related inputs into a case class
case class CreateOrderRequest(
  customerId: String,
  items: List[LineItemRequest],
  shippingAddress: Address
)

def createOrder(request: CreateOrderRequest): Order = ???
```

## Contract-First Documentation

```scala
/**
 * Places a new order for the given customer.
 *
 * @throws InvalidOrderException if items is empty.
 * @throws CustomerNotFoundException if customerId does not exist.
 * @return the created Order with a server-assigned id.
 */
def placeOrder(request: CreateOrderRequest): Order = ???
```

## Stable Versioning

```scala
// Add optional fields: backwards compatible
case class CreateOrderRequest(
  customerId: String,
  items: List[LineItemRequest],
  shippingAddress: Address,
  promoCode: Option[String] = None     // added later: optional, safe
)
```
