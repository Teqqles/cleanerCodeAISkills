# Dependency Injection: Java Examples

## Define the Interface in the Domain Layer

```java
// domain/order/OrderRepository.java
public interface OrderRepository {
    void save(Order order);
    Optional<Order> findById(String id);
}

// domain/order/Clock.java
public interface Clock {
    Instant now();
}
```

## Inject Through the Constructor

```java
// domain/order/OrderService.java
public class OrderService {
    private final OrderRepository orders;
    private final Clock clock;

    public OrderService(OrderRepository orders, Clock clock) {
        this.orders = orders;
        this.clock = clock;
    }

    public Order placeOrder(Cart cart) {
        Order order = Order.create(cart, clock.now());
        orders.save(order);
        return order;
    }
}
```

## In-Memory Fake for Tests

```java
// tests/fakes/InMemoryOrderRepository.java
public class InMemoryOrderRepository implements OrderRepository {
    private final Map<String, Order> store = new HashMap<>();

    @Override
    public void save(Order order) {
        store.put(order.getId(), order);
    }

    @Override
    public Optional<Order> findById(String id) {
        return Optional.ofNullable(store.get(id));
    }
}

public class FakeClock implements Clock {
    private final Instant fixed;

    public FakeClock(Instant fixed) { this.fixed = fixed; }

    @Override
    public Instant now() { return fixed; }
}
```

## Composition Root (Spring example)

```java
// AppConfig.java
@Configuration
public class AppConfig {
    @Bean
    public OrderService orderService(OrderRepository orders, Clock clock) {
        return new OrderService(orders, clock);
    }

    @Bean
    public Clock systemClock() {
        return Instant::now;
    }
}
```

Without a framework, wire manually in `main()`:

```java
public static void main(String[] args) {
    var orders = new PostgresOrderRepository(dataSource);
    var clock = Instant::now;
    var orderService = new OrderService(orders, clock);
}
```
