# Refactoring: Java Examples

## Extract Condition Into Named Method

```java
// Before
if (user.getSubscriptionEndDate().isAfter(Instant.now())
        && !user.getPlan().equals("free")
        && !user.isSuspended()) {
    showPremiumContent();
}

// After
private boolean hasActivePaidSubscription(User user) {
    return user.getSubscriptionEndDate().isAfter(Instant.now())
        && !user.getPlan().equals("free")
        && !user.isSuspended();
}

if (hasActivePaidSubscription(user)) {
    showPremiumContent();
}
```

## Replace Magic Values With Named Constants

```java
// Before
if (response.getStatus() == 429) {
    Thread.sleep(2000);
    retry(3);
}

// After
private static final int HTTP_TOO_MANY_REQUESTS = 429;
private static final long RETRY_DELAY_MS = 2_000L;
private static final int MAX_RETRIES = 3;

if (response.getStatus() == HTTP_TOO_MANY_REQUESTS) {
    Thread.sleep(RETRY_DELAY_MS);
    retry(MAX_RETRIES);
}
```

## Delete Dead Code

```java
// Before
public BigDecimal calculateShipping(Order order) {
    // BigDecimal legacyRate = new BigDecimal("4.99"); // old flat rate
    return order.totalWeightKg().multiply(new BigDecimal("1.5"));
}

// After
public BigDecimal calculateShipping(Order order) {
    return order.totalWeightKg().multiply(new BigDecimal("1.5"));
}
```

## Flatten Nesting With Early Returns

```java
// Before: 4 levels deep
public BigDecimal getInvoiceTotal(Invoice invoice) {
    if (invoice != null) {
        if (invoice.isPaid()) {
            if (!invoice.getLines().isEmpty()) {
                return invoice.getLines().stream()
                    .map(InvoiceLine::getAmount)
                    .reduce(BigDecimal.ZERO, BigDecimal::add);
            }
        }
    }
    return BigDecimal.ZERO;
}

// After: flat
public BigDecimal getInvoiceTotal(Invoice invoice) {
    if (invoice == null) return BigDecimal.ZERO;
    if (!invoice.isPaid()) return BigDecimal.ZERO;
    if (invoice.getLines().isEmpty()) return BigDecimal.ZERO;
    return invoice.getLines().stream()
        .map(InvoiceLine::getAmount)
        .reduce(BigDecimal.ZERO, BigDecimal::add);
}
```
