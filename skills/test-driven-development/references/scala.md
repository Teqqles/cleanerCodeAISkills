# Test-Driven Development: Scala Examples

Test framework: **ScalaTest** (`scalatest` dependency, `AnyFlatSpec` style)

## Red: Write the Failing Test First

```scala
// PriceCalculatorSpec.scala
class PriceCalculatorSpec extends AnyFlatSpec with Matchers {

  "PriceCalculator" should "apply percentage discount to base price" in {
    val calculator = new PriceCalculator
    val result = calculator.applyDiscount(BigDecimal(100), BigDecimal("0.1"))
    result shouldBe BigDecimal(90)
  }
}
// Run: sbt test: test fails, PriceCalculator does not exist yet
```

## Green: Minimum Code to Pass

```scala
// PriceCalculator.scala
class PriceCalculator {
  def applyDiscount(price: BigDecimal, discountRate: BigDecimal): BigDecimal =
    price - price * discountRate
}
// Run: sbt test: test passes
```

## Refactor: Improve Without Breaking

```scala
class PriceCalculator {
  def applyDiscount(price: BigDecimal, discountRate: BigDecimal): BigDecimal = {
    val discountAmount = discountAmountFor(price, discountRate)
    price - discountAmount
  }

  private def discountAmountFor(price: BigDecimal, rate: BigDecimal): BigDecimal =
    price * rate
}
// Run: sbt test: still passes
```

## Test Names Describe Behaviour

```scala
it should "return full price when discount rate is zero" in {
  new PriceCalculator().applyDiscount(BigDecimal(100), BigDecimal(0)) shouldBe BigDecimal(100)
}

it should "throw when discount rate exceeds 1" in {
  an[InvalidDiscountException] should be thrownBy {
    new PriceCalculator().applyDiscount(BigDecimal(100), BigDecimal("1.5"))
  }
}

it should "throw when price is negative" in {
  an[InvalidPriceException] should be thrownBy {
    new PriceCalculator().applyDiscount(BigDecimal(-1), BigDecimal("0.1"))
  }
}
```
