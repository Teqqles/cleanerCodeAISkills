# Code Style: Java

## Formatter

```bash
mvn spotless:apply
# or
java -jar google-java-format.jar --replace src/**/*.java
```

Typical config: `spotless` plugin in `pom.xml` or `build.gradle`

## Naming Conventions

```java
String userName = "Alice";              // camelCase: variables
void calculateTotal() {}                // camelCase: methods
class OrderService {}                   // PascalCase: classes
interface UserRepository {}             // PascalCase: interfaces
static final int MAX_RETRIES = 3;       // UPPER_SNAKE_CASE: constants
record UserId(String value) {}          // PascalCase: records
```

## Comments

```java
// Bad: describes what, not why
// Loop through users and sum balances
BigDecimal total = users.stream()
    .map(User::getBalance)
    .reduce(BigDecimal.ZERO, BigDecimal::add);

// Good: only when why is non-obvious
// Balances can be negative (credit accounts); sum without filtering is intentional
BigDecimal total = users.stream()
    .map(User::getBalance)
    .reduce(BigDecimal.ZERO, BigDecimal::add);
```

## Javadoc: Contracts, Not Implementation

```java
// Bad
/** This method loops through orders and calculates the total price */
public BigDecimal calculateTotal(List<Order> orders) { ... }

// Good
/**
 * Returns the sum of all order amounts.
 * Returns zero for an empty list.
 * Throws NullPointerException if orders is null.
 */
public BigDecimal calculateTotal(List<Order> orders) { ... }
```
