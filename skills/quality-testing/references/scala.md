# Quality Testing: Scala Examples

Test framework: **ScalaTest** (`AnyFlatSpec` with `Matchers`)

## R: Right (correct output)

```scala
"Order" should "return total from line items" in {
  val order = Order("o1", "c1", List(LineItem("book", BigDecimal(10)), LineItem("pen", BigDecimal(2))))
  order.total shouldBe BigDecimal(12)
}
```

## I: Inverse (roundtrip)

```scala
it should "restore original after serialise then deserialise" in {
  val order = Order.create(customerId = "c1", items = List(LineItem("book", BigDecimal(10))))
  Order.fromJson(order.toJson) shouldBe order
}
```

## H: Harmful (invalid inputs)

```scala
it should "throw when line item price is negative" in {
  an[InvalidPriceException] should be thrownBy LineItem("book", BigDecimal(-1))
}

it should "throw when item name is empty" in {
  an[InvalidItemException] should be thrownBy LineItem("", BigDecimal(10))
}
```

## T: Time (fake clock)

```scala
it should "mark order as expired after 30 minutes" in {
  val clock = FakeClock(Instant.parse("2024-01-01T09:00:00Z"))
  val order = Order.create(cart, clock)
  clock.advanceBy(Duration.ofMinutes(31))
  order.isExpired(clock.now()) shouldBe true
}
```

## B: Boundaries

```scala
it should "return zero total for empty order" in {
  Order("o1", "c1", Nil).total shouldBe BigDecimal(0)
}

it should "handle order with exactly one item" in {
  Order("o1", "c1", List(LineItem("book", BigDecimal(10)))).total shouldBe BigDecimal(10)
}
```

## E: Error Conditions

```scala
it should "throw OrderNotFoundException when id is missing" in {
  val repo = new InMemoryOrderRepository
  an[OrderNotFoundException] should be thrownBy repo.findById("missing")
}
```
