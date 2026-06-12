# API Design: Java Examples

## Explicit Inputs: No Hidden Dependencies

```java
// Bad: hidden dependency on thread-local session
public List<Product> getRecommendations() {
    User user = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    String region = System.getenv("REGION");
    return recommendationEngine.get(user, region);
}

// Good: all inputs declared
public List<Product> getRecommendations(
        String userId,
        String region,
        RecommendationEngine engine) {
    return engine.get(userId, region);
}
```

## Minimal Parameter Surface

```java
// Bad: seven parameters, caller must know internal structure
public Order createOrder(String customerId, List<String> skus, List<Integer> qtys,
                         List<BigDecimal> prices, String addressLine1, String addressLine2,
                         String city) { ... }

// Good: group related inputs into a record
public record CreateOrderRequest(
    String customerId,
    List<LineItemRequest> items,
    Address shippingAddress
) {}

public Order createOrder(CreateOrderRequest request) { ... }
```

## Contract-First Documentation

```java
/**
 * Places a new order for the given customer.
 *
 * @throws InvalidOrderException if items is empty.
 * @throws CustomerNotFoundException if customerId does not exist.
 * @return the created Order with a server-assigned id.
 */
public Order placeOrder(CreateOrderRequest request) { ... }
```

## Stable Versioning

```java
// Add optional fields: backwards compatible with existing callers
public record CreateOrderRequest(
    String customerId,
    List<LineItemRequest> items,
    Address shippingAddress,
    @Nullable String promoCode     // added later: nullable, safe
) {}
```
