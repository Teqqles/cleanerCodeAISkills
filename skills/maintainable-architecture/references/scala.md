# Maintainable Architecture: Scala Examples

## Layer Separation

```
src/main/scala/
├── domain/               # no framework imports: pure Scala
│   ├── order/
│   │   ├── Order.scala
│   │   ├── OrderRepository.scala  # trait (interface)
│   │   └── PlaceOrderUseCase.scala
│   └── payment/
│       ├── Payment.scala
│       └── PaymentGateway.scala   # trait (interface)
├── infrastructure/        # implements domain traits
│   ├── postgres/
│   │   └── PostgresOrderRepository.scala
│   └── stripe/
│       └── StripePaymentGateway.scala
└── api/                   # framework (Akka HTTP, Play, etc.)
    └── OrderRouter.scala
```

## Domain Types Have No Framework Dependencies

```scala
// domain/order/Order.scala: no Play, Slick, or framework imports
case class Order(id: String, customerId: String, items: List[LineItem]) {
  def total: BigDecimal = items.map(_.price).sum
}
```

## Map at the Boundary

```scala
// infrastructure/postgres/PostgresOrderRepository.scala
class PostgresOrderRepository(db: Database) extends OrderRepository {
  def findById(id: String): Option[Order] =
    db.run(orderQuery(id)).map(_.map(toDomain))   // map DB row to domain type here

  private def toDomain(row: OrderRow): Order =
    Order(row.id, row.customerId, row.items.map(toLineItem))
}
```

## Composition Over Inheritance

```scala
// Bad: subclassing for discount behaviour
class PremiumOrder(id: String, customerId: String, items: List[LineItem])
    extends Order(id, customerId, items) {
  override def total: BigDecimal = super.total * BigDecimal("0.8")
}

// Good: injected discount policy
case class Order(
  id: String,
  customerId: String,
  items: List[LineItem],
  discountPolicy: DiscountPolicy
) {
  def total: BigDecimal = discountPolicy.apply(items.map(_.price).sum)
}
```
