# Dependency Injection: Python Examples

## Define the Interface in the Domain Layer

```python
# domain/order/order_repository.py
from typing import Protocol

class OrderRepository(Protocol):
    def save(self, order: Order) -> None: ...
    def find_by_id(self, order_id: str) -> Order | None: ...

class Clock(Protocol):
    def now(self) -> datetime: ...
```

Python uses `Protocol` (structural typing) or `ABC` (nominal typing) for interfaces.

## Inject Through the Constructor

```python
# domain/order/order_service.py
class OrderService:
    def __init__(self, orders: OrderRepository, clock: Clock) -> None:
        self._orders = orders
        self._clock = clock

    def place_order(self, cart: Cart) -> Order:
        order = Order.create(cart, placed_at=self._clock.now())
        self._orders.save(order)
        return order
```

## In-Memory Fake for Tests

```python
# tests/fakes/in_memory_order_repository.py
class InMemoryOrderRepository:
    def __init__(self) -> None:
        self._store: dict[str, Order] = {}

    def save(self, order: Order) -> None:
        self._store[order.id] = order

    def find_by_id(self, order_id: str) -> Order | None:
        return self._store.get(order_id)

class FakeClock:
    def __init__(self, fixed: datetime) -> None:
        self._fixed = fixed

    def now(self) -> datetime:
        return self._fixed
```

## Composition Root

```python
# main.py: only place where concrete types appear together
order_service = OrderService(
    orders=PostgresOrderRepository(db),
    clock=SystemClock()
)
```
