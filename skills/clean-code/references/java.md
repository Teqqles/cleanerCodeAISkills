# Clean Code: Java Examples

## Naming

```java
// Bad
Date d = new Date();
List<User> arr = users.stream()
    .filter(u -> u.getA() > 18)
    .collect(toList());

// Good
Date currentDate = new Date();
List<User> adultUsers = users.stream()
    .filter(user -> user.getAge() > 18)
    .collect(toList());
```

## Early Returns

```java
// Bad
public BigDecimal getDiscount(User user) {
    if (user != null) {
        if (user.isPremium()) {
            if (user.getYearsActive() > 2) {
                return new BigDecimal("0.2");
            } else {
                return new BigDecimal("0.1");
            }
        }
    }
    return BigDecimal.ZERO;
}

// Good
public BigDecimal getDiscount(User user) {
    if (user == null) return BigDecimal.ZERO;
    if (!user.isPremium()) return BigDecimal.ZERO;
    return user.getYearsActive() > 2
        ? new BigDecimal("0.2")
        : new BigDecimal("0.1");
}
```

## Single-Purpose Functions

```java
// Bad: validates, saves, and sends email in one method
public void processUser(User user) {
    if (user.getEmail() == null) throw new IllegalArgumentException("missing email");
    userRepository.save(user);
    emailService.sendWelcome(user.getEmail());
}

// Good: each method does one thing
private void validateUser(User user) {
    if (user.getEmail() == null) throw new InvalidUserException("email is required");
}

public void registerUser(User user) {
    validateUser(user);
    userRepository.save(user);
    emailService.sendWelcome(user.getEmail());
}
```

## Domain-Centric Structure

```
src/main/java/
├── order/
│   ├── Order.java
│   ├── PlaceOrderUseCase.java
│   └── OrderTest.java
├── payment/
│   ├── Payment.java
│   └── ProcessPaymentUseCase.java
└── user/
    ├── User.java
    └── RegisterUserUseCase.java
```
