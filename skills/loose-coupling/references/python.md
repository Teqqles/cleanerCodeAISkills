# Loose Coupling: Python Examples

## Depend on Abstractions

```python
# Coupled: imports a concrete class
from infrastructure.postgres_user_store import PostgresUserStore

class UserService:
    def __init__(self):
        self.store = PostgresUserStore()

# Decoupled: depends on a Protocol the consumer defines
from typing import Protocol

class UserStore(Protocol):
    def find(self, user_id: str) -> User | None: ...
    def save(self, user: User) -> None: ...

class UserService:
    def __init__(self, store: UserStore):
        self._store = store
```

## Law of Demeter

```python
# Coupled: traverses the object graph
def get_shipping_city(order: Order) -> str:
    return order.get_customer().get_address().get_city()

# Decoupled: ask the object directly
def get_shipping_city(order: Order) -> str:
    return order.shipping_city()
```

## Tell, Don't Ask

```python
# Ask: caller interrogates then decides
def maybe_expire(session: Session) -> None:
    if time.time() - session.last_activity > session.timeout:
        session.expired = True
        notify_user(session.user_id)

# Tell: object owns the decision
class Session:
    def check_expiry(self, on_expired: Callable[[str], None]) -> None:
        if self._is_expired():
            on_expired(self.user_id)
```

## Interface Segregation

```python
# Broad: one protocol with everything
class DataStore(Protocol):
    def read(self, key: str) -> str: ...
    def write(self, key: str, value: str) -> None: ...
    def delete(self, key: str) -> None: ...
    def list_keys(self) -> list[str]: ...
    def watch(self, key: str, cb: Callable[[str], None]) -> None: ...

# Segregated: each consumer depends only on what it needs
class Readable(Protocol):
    def read(self, key: str) -> str: ...

class Writable(Protocol):
    def write(self, key: str, value: str) -> None: ...

class Watchable(Protocol):
    def watch(self, key: str, cb: Callable[[str], None]) -> None: ...
```

## Events Over Direct Calls

```python
# Coupled: OrderService knows its consumers
class OrderService:
    def __init__(self, inventory: InventoryService, email: EmailService, analytics: AnalyticsService):
        self._inventory = inventory
        self._email = email
        self._analytics = analytics

    def place(self, cart: Cart) -> Order:
        order = Order.create(cart)
        self._inventory.reserve(order.items)
        self._email.send_confirmation(order)
        self._analytics.track("order_placed", order)
        return order

# Decoupled: emit, let subscribers handle it
class OrderService:
    def __init__(self):
        self.order_placed = EventEmitter[Order]()

    def place(self, cart: Cart) -> Order:
        order = Order.create(cart)
        self.order_placed.emit(order)
        return order

# Wired at the composition root
order_service.order_placed.subscribe(lambda o: inventory.reserve(o.items))
order_service.order_placed.subscribe(lambda o: email.send_confirmation(o))
```

## Extension Without Modification

```python
# Coupled: if/elif chain grows with every format
def export_report(report: Report, fmt: str) -> str:
    if fmt == "csv":
        return to_csv(report)
    elif fmt == "json":
        return to_json(report)
    elif fmt == "xml":
        return to_xml(report)
    raise ValueError(f"Unknown format: {fmt}")

# Decoupled: new formats are new classes
class ReportExporter(Protocol):
    def export(self, report: Report) -> str: ...

class CsvExporter:
    def export(self, report: Report) -> str:
        return to_csv(report)

class JsonExporter:
    def export(self, report: Report) -> str:
        return to_json(report)
```
