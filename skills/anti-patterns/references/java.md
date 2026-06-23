# Anti-Patterns and Code Smells: Java Examples

## God Class

```java
// Smell: OrderManager handles unrelated responsibilities
public class OrderManager {
    public Order placeOrder(Cart cart) { /* ... */ }
    public void cancelOrder(String id) { /* ... */ }
    public int checkInventory(String sku) { /* ... */ }
    public void reserveStock(String sku, int qty) { /* ... */ }
    public void sendConfirmationEmail(Order order) { /* ... */ }
    public Report generateMonthlyReport() { /* ... */ }
}

// Fix: focused classes
public class OrderService {
    public Order placeOrder(Cart cart) { /* ... */ }
    public void cancelOrder(String id) { /* ... */ }
}

public class InventoryService {
    public int checkStock(String sku) { /* ... */ }
    public void reserve(String sku, int qty) { /* ... */ }
}

public class OrderNotifications {
    public void sendConfirmation(Order order) { /* ... */ }
}
```

## Feature Envy

```java
// Smell: reaches into customer's internals
public String formatInvoiceAddress(Order order) {
    Customer c = order.getCustomer();
    Address a = c.getAddress();
    return c.getFirstName() + " " + c.getLastName() + "\n"
         + a.getStreet() + "\n"
         + a.getCity() + ", " + a.getPostcode();
}

// Fix: behaviour on the object that owns the data
public class Customer {
    public String formatAddress() {
        return firstName + " " + lastName + "\n"
             + address.getStreet() + "\n"
             + address.getCity() + ", " + address.getPostcode();
    }
}
```

## Primitive Obsession

```java
// Smell: bare strings with validation scattered across the codebase
public User createUser(String email, String phone, String role) {
    if (!email.contains("@")) throw new IllegalArgumentException("invalid email");
    return new User(email, phone, role);
}

// Fix: value types that enforce validity
public record EmailAddress(String value) {
    public EmailAddress {
        if (!value.contains("@")) throw new InvalidEmailException(value);
        value = value.toLowerCase();
    }
}

public record PhoneNumber(String value) {
    public PhoneNumber {
        String digits = value.replaceAll("\\D", "");
        if (digits.length() < 10) throw new InvalidPhoneException(value);
        value = digits;
    }
}

public User createUser(EmailAddress email, PhoneNumber phone, UserRole role) {
    return new User(email, phone, role);
}
```

## Flag Arguments

```java
// Smell: boolean toggles behaviour
public void sendNotification(User user, boolean urgent) {
    if (urgent) {
        sms.send(user.getPhone(), buildUrgentMessage(user));
    } else {
        email.send(user.getEmail(), buildStandardMessage(user));
    }
}

// Fix: named methods
public void sendUrgentNotification(User user) {
    sms.send(user.getPhone(), buildUrgentMessage(user));
}

public void sendStandardNotification(User user) {
    email.send(user.getEmail(), buildStandardMessage(user));
}
```

## Temporal Coupling

```java
// Smell: must call init() before process()
public class ReportGenerator {
    public void init(Config config) { /* ... */ }
    public void loadData() { /* ... */ }
    public Report process() { /* ... */ }
}

// Fix: enforce the sequence through the API
public class ReportGenerator {
    public static Report generate(Config config) {
        var data = loadData(config);
        return processData(data);
    }
}
```

## Data Clumps

```java
// Smell: lat/lon always travel together
public double calculateDistance(double lat1, double lon1, double lat2, double lon2) { /* ... */ }
public String formatLocation(double lat, double lon) { /* ... */ }

// Fix: named record
public record Coordinate(double latitude, double longitude) {}

public double calculateDistance(Coordinate from, Coordinate to) { /* ... */ }
public String formatLocation(Coordinate coord) { /* ... */ }
```
