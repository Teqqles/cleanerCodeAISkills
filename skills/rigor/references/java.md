# Engineering Rigor: Java Examples

## Typed Errors: No Silent Failures

```java
// Bad: exception swallowed
public void chargeCustomer(Order order) {
    try {
        paymentGateway.charge(order);
    } catch (Exception e) {
        // silently ignored
    }
}

// Good: typed result, failure surfaced
public sealed interface ChargeResult {
    record Success(String chargeId) implements ChargeResult {}
    record Failure(String reason)   implements ChargeResult {}
}

public ChargeResult chargeCustomer(Order order) {
    try {
        String chargeId = paymentGateway.charge(order);
        return new ChargeResult.Success(chargeId);
    } catch (PaymentGatewayException e) {
        logger.error("charge_failed orderId={}", order.getId(), e);
        return new ChargeResult.Failure("gateway_error");
    }
}
```

## Explicit Precondition Enforcement

```java
// Bad: NullPointerException or wrong result later
public BigDecimal applyDiscount(BigDecimal price, BigDecimal rate) {
    return price.subtract(price.multiply(rate));
}

// Good: preconditions checked at the boundary
public BigDecimal applyDiscount(BigDecimal price, BigDecimal rate) {
    if (price.compareTo(BigDecimal.ZERO) < 0)
        throw new InvalidPriceException("price must be non-negative, got " + price);
    if (rate.compareTo(BigDecimal.ZERO) < 0 || rate.compareTo(BigDecimal.ONE) > 0)
        throw new InvalidDiscountException("rate must be 0-1, got " + rate);
    return price.subtract(price.multiply(rate));
}
```

## Structured Logging

```java
// Bad: unstructured, hard to query
System.out.println("order placed for user " + userId);

// Good: structured with SLF4J
logger.info("order_placed orderId={} customerId={} total={} itemCount={}",
    order.getId(), order.getCustomerId(), order.total(), order.getItems().size());
```

## Environment-Driven Configuration

```java
// Bad: hardcoded
private static final String DB_URL = "postgres://localhost:5432/mydb";

// Good: from environment
String dbUrl = System.getenv("DATABASE_URL");
if (dbUrl == null || dbUrl.isBlank()) {
    throw new ConfigurationException("DATABASE_URL is required");
}
```
