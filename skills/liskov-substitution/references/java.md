# Liskov Substitution: Java Examples

## Violation: Subtype That Throws on Inherited Methods

```java
public interface Collection<T> {
    void add(T item);
    T get(int index);
    int size();
}

public class ReadOnlyList<T> implements Collection<T> {
    @Override
    public void add(T item) {
        throw new UnsupportedOperationException("Cannot add to read-only list");  // violates LSP
    }

    @Override
    public T get(int index) { /* ... */ }

    @Override
    public int size() { /* ... */ }
}
```

Fix: separate interfaces.

```java
public interface Readable<T> {
    T get(int index);
    int size();
}

public interface Writable<T> extends Readable<T> {
    void add(T item);
}

public class ReadOnlyList<T> implements Readable<T> {
    @Override
    public T get(int index) { /* ... */ }

    @Override
    public int size() { /* ... */ }
}
```

## Violation: Subtype That Strengthens Preconditions

```java
public interface Discount {
    double apply(double amount);
}

public class PercentDiscount implements Discount {
    @Override
    public double apply(double amount) {
        return amount * 0.9;
    }
}

public class MinimumOrderDiscount implements Discount {
    @Override
    public double apply(double amount) {
        if (amount < 50) throw new IllegalArgumentException("Order too small");  // violates LSP
        return amount * 0.8;
    }
}
```

Fix: accept all inputs the base contract accepts.

```java
public class MinimumOrderDiscount implements Discount {
    @Override
    public double apply(double amount) {
        if (amount < 50) return amount;  // no discount, no exception
        return amount * 0.8;
    }
}
```

## Contract Tests Proving LSP

```java
public abstract class DiscountContractTest {
    protected abstract Discount createDiscount();

    @Test
    void appliestoAnyNonNegativeAmount() {
        var discount = createDiscount();
        assertThat(discount.apply(0)).isGreaterThanOrEqualTo(0);
        assertThat(discount.apply(10)).isGreaterThanOrEqualTo(0);
        assertThat(discount.apply(1000)).isGreaterThanOrEqualTo(0);
    }

    @Test
    void neverReturnsMoreThanInput() {
        var discount = createDiscount();
        assertThat(discount.apply(100)).isLessThanOrEqualTo(100);
    }
}

// One test class per implementation
public class PercentDiscountTest extends DiscountContractTest {
    @Override
    protected Discount createDiscount() { return new PercentDiscount(); }
}

public class MinimumOrderDiscountTest extends DiscountContractTest {
    @Override
    protected Discount createDiscount() { return new MinimumOrderDiscount(); }
}
```

## Violation: instanceof Checks in Calling Code

```java
// Caller forced to inspect concrete type
public String render(Shape shape) {
    if (shape instanceof Circle c) {
        return "circle(" + c.getRadius() + ")";
    } else if (shape instanceof Rectangle r) {
        return "rect(" + r.getWidth() + ", " + r.getHeight() + ")";
    }
    throw new IllegalArgumentException("Unknown shape");
}

// Fixed: polymorphism through the interface
public interface Shape {
    String render();
}

public record Circle(double radius) implements Shape {
    @Override
    public String render() { return "circle(" + radius + ")"; }
}

public record Rectangle(double width, double height) implements Shape {
    @Override
    public String render() { return "rect(" + width + ", " + height + ")"; }
}

public String render(Shape shape) {
    return shape.render();
}
```
