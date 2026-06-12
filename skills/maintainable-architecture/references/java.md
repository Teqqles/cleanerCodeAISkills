# Maintainable Architecture: Java Examples

## Layer Separation

```
src/main/java/
├── domain/               # no framework imports: plain Java
│   ├── order/
│   │   ├── Order.java
│   │   ├── OrderRepository.java   # interface
│   │   └── PlaceOrderUseCase.java
│   └── payment/
│       ├── Payment.java
│       └── PaymentGateway.java    # interface
├── infrastructure/        # implements domain interfaces
│   ├── postgres/
│   │   └── PostgresOrderRepository.java
│   └── stripe/
│       └── StripePaymentGateway.java
└── api/                   # framework (Spring MVC, etc.)
    └── OrderController.java
```

## Domain Types Have No Framework Dependencies

```java
// domain/order/Order.java: no Spring, JPA, or framework annotations
public record Order(String id, String customerId, List<LineItem> items) {

    public BigDecimal total() {
        return items.stream()
            .map(LineItem::price)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

## Map at the Boundary

```java
// infrastructure/postgres/PostgresOrderRepository.java
public class PostgresOrderRepository implements OrderRepository {

    @Override
    public Optional<Order> findById(String id) {
        return db.query("SELECT * FROM orders WHERE id = ?", id)
            .stream()
            .findFirst()
            .map(this::toDomain);    // map JPA entity to domain type here
    }

    private Order toDomain(OrderEntity entity) {
        return new Order(entity.getId(), entity.getCustomerId(),
            entity.getItems().stream().map(this::toLineItem).toList());
    }
}
```

## Composition Over Inheritance

```java
// Bad: subclassing for discount behaviour
class PremiumOrder extends Order {
    @Override public BigDecimal total() { return super.total().multiply(new BigDecimal("0.8")); }
}

// Good: injected discount policy
public class Order {
    private final List<LineItem> items;
    private final DiscountPolicy discountPolicy;

    public Order(List<LineItem> items, DiscountPolicy discountPolicy) {
        this.items = items;
        this.discountPolicy = discountPolicy;
    }

    public BigDecimal total() {
        BigDecimal subtotal = items.stream()
            .map(LineItem::price)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        return discountPolicy.apply(subtotal);
    }
}
```
