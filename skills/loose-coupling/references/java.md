# Loose Coupling: Java Examples

## Depend on Abstractions

```java
// Coupled: imports a concrete class
import infrastructure.PostgresUserStore;

public class UserService {
    private final PostgresUserStore store = new PostgresUserStore();
}

// Decoupled: depends on an interface
public interface UserStore {
    Optional<User> find(String id);
    void save(User user);
}

public class UserService {
    private final UserStore store;

    public UserService(UserStore store) {
        this.store = store;
    }
}
```

## Law of Demeter

```java
// Coupled: traverses the object graph
public String getShippingCity(Order order) {
    return order.getCustomer().getAddress().getCity();
}

// Decoupled: ask the object directly
public String getShippingCity(Order order) {
    return order.shippingCity();
}
```

## Tell, Don't Ask

```java
// Ask: caller interrogates then decides
public void maybeExpire(Session session) {
    if (System.currentTimeMillis() - session.getLastActivity() > session.getTimeout()) {
        session.setExpired(true);
        notifyUser(session.getUserId());
    }
}

// Tell: object owns the decision
public class Session {
    public void checkExpiry(Consumer<String> onExpired) {
        if (isExpired()) {
            onExpired.accept(userId);
        }
    }
}
```

## Interface Segregation

```java
// Broad: one interface with everything
public interface DataStore {
    String read(String key);
    void write(String key, String value);
    void delete(String key);
    List<String> listKeys();
    void watch(String key, Consumer<String> callback);
}

// Segregated
public interface Readable {
    String read(String key);
}

public interface Writable {
    void write(String key, String value);
}

public interface Watchable {
    void watch(String key, Consumer<String> callback);
}
```

## Events Over Direct Calls

```java
// Coupled: OrderService knows all its consumers
public class OrderService {
    private final InventoryService inventory;
    private final EmailService email;
    private final AnalyticsService analytics;

    public Order place(Cart cart) {
        Order order = Order.create(cart);
        inventory.reserve(order.getItems());
        email.sendConfirmation(order);
        analytics.track("order_placed", order);
        return order;
    }
}

// Decoupled: emit events, consumers subscribe
public class OrderService {
    private final EventEmitter<Order> orderPlaced = new EventEmitter<>();

    public EventEmitter<Order> onOrderPlaced() { return orderPlaced; }

    public Order place(Cart cart) {
        Order order = Order.create(cart);
        orderPlaced.emit(order);
        return order;
    }
}

// Composition root
orderService.onOrderPlaced().subscribe(order -> inventory.reserve(order.getItems()));
orderService.onOrderPlaced().subscribe(order -> email.sendConfirmation(order));
orderService.onOrderPlaced().subscribe(order -> analytics.track("order_placed", order));
```

## Extension Without Modification

```java
// Coupled: switch grows with every format
public String exportReport(Report report, String format) {
    return switch (format) {
        case "csv" -> toCsv(report);
        case "json" -> toJson(report);
        case "xml" -> toXml(report);
        default -> throw new IllegalArgumentException("Unknown format: " + format);
    };
}

// Decoupled: new formats are new implementations
public interface ReportExporter {
    String export(Report report);
}

public class CsvExporter implements ReportExporter {
    @Override
    public String export(Report report) { return toCsv(report); }
}

public class JsonExporter implements ReportExporter {
    @Override
    public String export(Report report) { return toJson(report); }
}
```
