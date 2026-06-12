# Test-Driven Development: Java Examples

Test framework: **JUnit 5** with **AssertJ** (`junit-jupiter`, `assertj-core`)

## Red: Write the Failing Test First

```java
// PriceCalculatorTest.java
class PriceCalculatorTest {
    @Test
    void appliesPercentageDiscountToBasePrice() {
        var calculator = new PriceCalculator();
        var result = calculator.applyDiscount(
            new BigDecimal("100"), new BigDecimal("0.1")
        );
        assertThat(result).isEqualByComparingTo(new BigDecimal("90.0"));
    }
}
// Run: ./mvnw test: test fails, PriceCalculator does not exist yet
```

## Green: Minimum Code to Pass

```java
// PriceCalculator.java
public class PriceCalculator {
    public BigDecimal applyDiscount(BigDecimal price, BigDecimal discountRate) {
        return price.subtract(price.multiply(discountRate));
    }
}
// Run: ./mvnw test: test passes
```

## Refactor: Improve Without Breaking

```java
public class PriceCalculator {
    public BigDecimal applyDiscount(BigDecimal price, BigDecimal discountRate) {
        BigDecimal discountAmount = discountAmountFor(price, discountRate);
        return price.subtract(discountAmount);
    }

    private BigDecimal discountAmountFor(BigDecimal price, BigDecimal rate) {
        return price.multiply(rate);
    }
}
// Run: ./mvnw test: still passes
```

## Test Names Describe Behaviour

```java
@Test void returnsFullPriceWhenDiscountRateIsZero() {
    assertThat(new PriceCalculator().applyDiscount(
        new BigDecimal("100"), BigDecimal.ZERO)
    ).isEqualByComparingTo(new BigDecimal("100"));
}

@Test void throwsWhenDiscountRateExceedsOne() {
    assertThatThrownBy(() ->
        new PriceCalculator().applyDiscount(new BigDecimal("100"), new BigDecimal("1.5"))
    ).isInstanceOf(InvalidDiscountException.class);
}

@Test void throwsWhenPriceIsNegative() {
    assertThatThrownBy(() ->
        new PriceCalculator().applyDiscount(new BigDecimal("-1"), new BigDecimal("0.1"))
    ).isInstanceOf(InvalidPriceException.class);
}
```
