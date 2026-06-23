# Design Patterns: Java Examples

## Strategy

```java
public interface PricingStrategy {
    BigDecimal calculatePrice(BigDecimal basePrice, int quantity);
}

public class StandardPricing implements PricingStrategy {
    @Override
    public BigDecimal calculatePrice(BigDecimal basePrice, int quantity) {
        return basePrice.multiply(BigDecimal.valueOf(quantity));
    }
}

public class BulkPricing implements PricingStrategy {
    @Override
    public BigDecimal calculatePrice(BigDecimal basePrice, int quantity) {
        BigDecimal discount = quantity > 100 ? new BigDecimal("0.8")
                            : quantity > 50 ? new BigDecimal("0.9")
                            : BigDecimal.ONE;
        return basePrice.multiply(BigDecimal.valueOf(quantity)).multiply(discount);
    }
}

public class OrderCalculator {
    private final PricingStrategy pricing;

    public OrderCalculator(PricingStrategy pricing) {
        this.pricing = pricing;
    }

    public BigDecimal total(BigDecimal basePrice, int quantity) {
        return pricing.calculatePrice(basePrice, quantity);
    }
}
```

## Observer

```java
public class EventEmitter<T> {
    private final List<Consumer<T>> listeners = new ArrayList<>();

    public Runnable subscribe(Consumer<T> listener) {
        listeners.add(listener);
        return () -> listeners.remove(listener);
    }

    public void emit(T event) {
        listeners.forEach(listener -> listener.accept(event));
    }
}

public class OrderService {
    private final EventEmitter<Order> orderPlaced = new EventEmitter<>();

    public EventEmitter<Order> onOrderPlaced() { return orderPlaced; }

    public Order place(Cart cart) {
        Order order = Order.create(cart);
        orderPlaced.emit(order);
        return order;
    }
}
```

## Factory

```java
public interface NotificationChannel {
    void send(String message, String recipient);
}

public class NotificationChannelFactory {
    public NotificationChannel create(String type) {
        return switch (type) {
            case "email" -> new EmailChannel();
            case "sms" -> new SmsChannel();
            case "push" -> new PushChannel();
            default -> throw new IllegalArgumentException("Unknown channel: " + type);
        };
    }
}
```

## Decorator

```java
public interface HttpClient {
    Response get(String url) throws IOException;
}

public class LoggingHttpClient implements HttpClient {
    private final HttpClient inner;
    private final Logger logger;

    public LoggingHttpClient(HttpClient inner, Logger logger) {
        this.inner = inner;
        this.logger = logger;
    }

    @Override
    public Response get(String url) throws IOException {
        logger.info("GET " + url);
        Response response = inner.get(url);
        logger.info(url + " -> " + response.status());
        return response;
    }
}

public class RetryingHttpClient implements HttpClient {
    private final HttpClient inner;
    private final int maxRetries;

    public RetryingHttpClient(HttpClient inner, int maxRetries) {
        this.inner = inner;
        this.maxRetries = maxRetries;
    }

    @Override
    public Response get(String url) throws IOException {
        IOException lastError = null;
        for (int i = 0; i <= maxRetries; i++) {
            try { return inner.get(url); }
            catch (IOException e) { lastError = e; }
        }
        throw lastError;
    }
}

// Compose
HttpClient client = new RetryingHttpClient(
    new LoggingHttpClient(new JdkHttpClient(), logger),
    3
);
```

## Adapter

```java
public class StripeAdapter implements PaymentGateway {
    private final StripeClient stripe;

    public StripeAdapter(StripeClient stripe) {
        this.stripe = stripe;
    }

    @Override
    public PaymentResult charge(BigDecimal amount, String currency) {
        PaymentIntent intent = stripe.paymentIntents().create(
            amount.multiply(new BigDecimal("100")).intValue(),
            currency
        );
        String status = "succeeded".equals(intent.getStatus()) ? "ok" : "failed";
        return new PaymentResult(intent.getId(), status);
    }
}
```
