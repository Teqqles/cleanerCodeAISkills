# Liskov Substitution: Scala Examples

## Violation: Subtype That Throws on Inherited Methods

```scala
trait Collection[T] {
  def add(item: T): Unit
  def get(index: Int): T
  def size: Int
}

class ReadOnlyList[T] extends Collection[T] {
  def add(item: T): Unit =
    throw new UnsupportedOperationException("Cannot add")  // violates LSP

  def get(index: Int): T = ???
  def size: Int = ???
}
```

Fix: separate traits.

```scala
trait Readable[T] {
  def get(index: Int): T
  def size: Int
}

trait Writable[T] extends Readable[T] {
  def add(item: T): Unit
}

class ReadOnlyList[T](items: Vector[T]) extends Readable[T] {
  def get(index: Int): T = items(index)
  def size: Int = items.length
}
```

## Violation: Subtype That Strengthens Preconditions

```scala
trait Discount {
  def apply(amount: BigDecimal): BigDecimal
}

class PercentDiscount extends Discount {
  def apply(amount: BigDecimal): BigDecimal = amount * 0.9
}

class MinimumOrderDiscount extends Discount {
  def apply(amount: BigDecimal): BigDecimal = {
    require(amount >= 50, "Order too small")  // strengthened precondition
    amount * 0.8
  }
}
```

Fix: accept all inputs the base contract accepts.

```scala
class MinimumOrderDiscount extends Discount {
  def apply(amount: BigDecimal): BigDecimal =
    if (amount < 50) amount  // no discount, no exception
    else amount * 0.8
}
```

## Contract Tests Proving LSP

```scala
import org.scalatest.funsuite.AnyFunSuite

abstract class DiscountContractTest extends AnyFunSuite {
  def createDiscount(): Discount

  test("applies to any non-negative amount") {
    val discount = createDiscount()
    assert(discount.apply(0) >= 0)
    assert(discount.apply(10) >= 0)
    assert(discount.apply(1000) >= 0)
  }

  test("never returns more than input") {
    val discount = createDiscount()
    assert(discount.apply(100) <= 100)
  }
}

class PercentDiscountTest extends DiscountContractTest {
  def createDiscount(): Discount = new PercentDiscount
}

class MinimumOrderDiscountTest extends DiscountContractTest {
  def createDiscount(): Discount = new MinimumOrderDiscount
}
```

## Violation: Pattern Matching on Concrete Types

```scala
// Caller forced to inspect concrete type
def render(shape: Shape): String = shape match {
  case c: Circle    => s"circle(${c.radius})"
  case r: Rectangle => s"rect(${r.width}, ${r.height})"
  case _            => throw new IllegalArgumentException("Unknown shape")
}

// Fixed: polymorphism through the trait
trait Shape {
  def render: String
}

case class Circle(radius: Double) extends Shape {
  def render: String = s"circle($radius)"
}

case class Rectangle(width: Double, height: Double) extends Shape {
  def render: String = s"rect($width, $height)"
}

def render(shape: Shape): String = shape.render
```

Note: Scala sealed traits with exhaustive pattern matching are a valid alternative when the set of subtypes is fixed and known at compile time. LSP violations occur when the match is non-exhaustive or when subtypes silently break expected behaviour.
