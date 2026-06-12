# Quality Testing: Java Examples

Test framework: **JUnit 5** with **AssertJ**

## R: Right (correct output)

```java
@Test void returnsOrderTotalFromLineItems() {
    var order = new Order(List.of(new LineItem("book", new BigDecimal("10")),
                                  new LineItem("pen",  new BigDecimal("2"))));
    assertThat(order.total()).isEqualByComparingTo(new BigDecimal("12"));
}
```

## I: Inverse (roundtrip)

```java
@Test void serialisingThenDeserialisingReturnsOriginal() {
    var order = Order.create("c1", List.of(new LineItem("book", new BigDecimal("10"))));
    var restored = Order.fromJson(order.toJson());
    assertThat(restored).isEqualTo(order);
}
```

## H: Harmful (invalid inputs)

```java
@Test void throwsWhenLineItemPriceIsNegative() {
    assertThatThrownBy(() -> new LineItem("book", new BigDecimal("-1")))
        .isInstanceOf(InvalidPriceException.class);
}

@Test void throwsWhenItemNameIsEmpty() {
    assertThatThrownBy(() -> new LineItem("", new BigDecimal("10")))
        .isInstanceOf(InvalidItemException.class);
}
```

## T: Time (fake clock)

```java
@Test void marksOrderAsExpiredAfter30Minutes() {
    var clock = new FakeClock(Instant.parse("2024-01-01T09:00:00Z"));
    var order = Order.create(cart, clock);
    clock.advanceBy(Duration.ofMinutes(31));
    assertThat(order.isExpired(clock.now())).isTrue();
}
```

## B: Boundaries

```java
@Test void returnsZeroTotalForEmptyOrder() {
    assertThat(new Order(List.of()).total()).isEqualByComparingTo(BigDecimal.ZERO);
}

@Test void handlesOrderWithExactlyOneItem() {
    var order = new Order(List.of(new LineItem("book", new BigDecimal("10"))));
    assertThat(order.total()).isEqualByComparingTo(new BigDecimal("10"));
}
```

## E: Error Conditions

```java
@Test void throwsOrderNotFoundWhenIdIsMissing() {
    var repo = new InMemoryOrderRepository();
    assertThatThrownBy(() -> repo.findById("missing"))
        .isInstanceOf(OrderNotFoundException.class);
}
```
