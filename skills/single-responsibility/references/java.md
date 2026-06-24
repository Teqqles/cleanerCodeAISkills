# Single Responsibility: Java Examples

## Splitting a Multi-Responsibility Class

```java
// Violated: UserService handles persistence, validation, and notifications
public class UserService {
    private final Database db;
    private final Mailer mailer;

    public UserService(Database db, Mailer mailer) {
        this.db = db;
        this.mailer = mailer;
    }

    public User register(UserInput data) {
        if (!data.getEmail().contains("@"))
            throw new InvalidInputException("bad email");
        User user = new User(data.getEmail(), data.getName());
        db.save(user);
        mailer.send(user.getEmail(), "Welcome!");
        return user;
    }
}

// Fixed: each class has one reason to change
public class UserValidator {
    public void validate(UserInput data) {
        if (!data.getEmail().contains("@"))
            throw new InvalidInputException("bad email");
    }
}

public class UserRepository {
    private final Database db;

    public UserRepository(Database db) { this.db = db; }

    public void save(User user) { db.save(user); }
}

public class WelcomeNotifier {
    private final Mailer mailer;

    public WelcomeNotifier(Mailer mailer) { this.mailer = mailer; }

    public void notify(User user) { mailer.send(user.getEmail(), "Welcome!"); }
}

public class RegisterUser {
    private final UserValidator validator;
    private final UserRepository repo;
    private final WelcomeNotifier notifier;

    public RegisterUser(UserValidator validator, UserRepository repo, WelcomeNotifier notifier) {
        this.validator = validator;
        this.repo = repo;
        this.notifier = notifier;
    }

    public User execute(UserInput data) {
        validator.validate(data);
        User user = new User(data.getEmail(), data.getName());
        repo.save(user);
        notifier.notify(user);
        return user;
    }
}
```

## Function-Level SRP: Separate Orchestration from Detail

```java
// Violated: mixes calculation with formatting
public String generateReport(List<Order> orders) {
    var total = orders.stream().mapToDouble(Order::getAmount).sum();
    var lines = orders.stream()
        .map(o -> o.getId() + ": " + String.format("%.2f", o.getAmount()))
        .collect(Collectors.joining("\n"));
    return "Report\n" + lines + "\nTotal: " + String.format("%.2f", total);
}

// Fixed
private double calculateTotal(List<Order> orders) {
    return orders.stream().mapToDouble(Order::getAmount).sum();
}

private String formatReport(List<Order> orders, double total) {
    var lines = orders.stream()
        .map(o -> o.getId() + ": " + String.format("%.2f", o.getAmount()))
        .collect(Collectors.joining("\n"));
    return "Report\n" + lines + "\nTotal: " + String.format("%.2f", total);
}

public String generateReport(List<Order> orders) {
    double total = calculateTotal(orders);
    return formatReport(orders, total);
}
```

## Package-Level SRP

```
// Violated: order package includes payment logic
com.example.order/
├── Order.java
├── PlaceOrder.java
├── ChargeCard.java       // payment concern
└── RefundPayment.java    // payment concern

// Fixed: separate packages
com.example.order/
├── Order.java
└── PlaceOrder.java

com.example.payment/
├── ChargeCard.java
└── RefundPayment.java
```
