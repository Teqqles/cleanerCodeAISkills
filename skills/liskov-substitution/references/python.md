# Liskov Substitution: Python Examples

## Violation: Subtype That Throws on Inherited Methods

```python
from typing import Protocol

class Collection(Protocol):
    def add(self, item: object) -> None: ...
    def get(self, index: int) -> object: ...
    def size(self) -> int: ...

class ReadOnlyList:
    def add(self, item: object) -> None:
        raise NotImplementedError("Cannot add to read-only list")  # violates LSP

    def get(self, index: int) -> object: ...
    def size(self) -> int: ...
```

Fix: separate protocols.

```python
class Readable(Protocol):
    def get(self, index: int) -> object: ...
    def size(self) -> int: ...

class Writable(Protocol):
    def add(self, item: object) -> None: ...

class ReadOnlyList:
    def get(self, index: int) -> object: ...
    def size(self) -> int: ...
```

## Violation: Subtype That Strengthens Preconditions

```python
class Discount(Protocol):
    def apply(self, amount: float) -> float: ...

class PercentDiscount:
    def apply(self, amount: float) -> float:
        return amount * 0.9

class MinimumOrderDiscount:
    def apply(self, amount: float) -> float:
        if amount < 50:
            raise ValueError("Order too small")  # strengthened precondition
        return amount * 0.8
```

Fix: accept all inputs the base contract accepts.

```python
class MinimumOrderDiscount:
    def apply(self, amount: float) -> float:
        if amount < 50:
            return amount  # no discount, no exception
        return amount * 0.8
```

## Contract Tests Proving LSP

```python
import pytest

def discount_contract_tests(create_discount):
    discount = create_discount()

    def test_applies_to_any_non_negative_amount():
        assert discount.apply(0) >= 0
        assert discount.apply(10) >= 0
        assert discount.apply(1000) >= 0

    def test_never_returns_more_than_input():
        assert discount.apply(100) <= 100

    test_applies_to_any_non_negative_amount()
    test_never_returns_more_than_input()

# Run against every implementation
discount_contract_tests(lambda: PercentDiscount())
discount_contract_tests(lambda: MinimumOrderDiscount())
discount_contract_tests(lambda: BulkDiscount())
```

## Violation: isinstance Checks in Calling Code

```python
# Caller forced to inspect concrete type
def render(shape: Shape) -> str:
    if isinstance(shape, Circle):
        return f"circle({shape.radius})"
    elif isinstance(shape, Rectangle):
        return f"rect({shape.width}, {shape.height})"
    raise ValueError("Unknown shape")

# Fixed: polymorphism through the protocol
class Shape(Protocol):
    def render(self) -> str: ...

class Circle:
    def __init__(self, radius: float):
        self.radius = radius

    def render(self) -> str:
        return f"circle({self.radius})"

class Rectangle:
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    def render(self) -> str:
        return f"rect({self.width}, {self.height})"

def render(shape: Shape) -> str:
    return shape.render()
```
