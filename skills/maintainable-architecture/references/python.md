# Maintainable Architecture: Python Examples

## Layer Separation

```
src/
├── domain/               # no framework imports: pure Python
│   ├── order/
│   │   ├── order.py
│   │   ├── order_repository.py    # Protocol (interface)
│   │   └── place_order.py
│   └── payment/
│       ├── payment.py
│       └── payment_gateway.py     # Protocol (interface)
├── infrastructure/        # implements domain protocols
│   ├── postgres/
│   │   └── postgres_order_repository.py
│   └── stripe/
│       └── stripe_payment_gateway.py
└── api/                   # framework (FastAPI, Django, Flask)
    └── order_router.py
```

## Domain Types Have No Framework Dependencies

```python
# domain/order/order.py: no imports from SQLAlchemy, FastAPI, or any framework
from dataclasses import dataclass
from decimal import Decimal

@dataclass(frozen=True)
class Order:
    id: str
    customer_id: str
    items: tuple[LineItem, ...]

    def total(self) -> Decimal:
        return sum(item.price for item in self.items)
```

## Map at the Boundary

```python
# infrastructure/postgres/postgres_order_repository.py
class PostgresOrderRepository:
    def find_by_id(self, order_id: str) -> Order | None:
        row = self._db.execute('SELECT * FROM orders WHERE id = %s', (order_id,))
        if not row:
            return None
        return self._to_domain(row)    # map DB row to domain type here

    def _to_domain(self, row: dict) -> Order:
        return Order(
            id=row['id'],
            customer_id=row['customer_id'],
            items=tuple(to_line_item(r) for r in row['items'])
        )
```

## Composition Over Inheritance

```python
# Bad: subclassing for discount behaviour
class PremiumOrder(Order):
    def total(self) -> Decimal:
        return super().total() * Decimal('0.8')

# Good: injected discount policy
@dataclass(frozen=True)
class Order:
    items: tuple[LineItem, ...]
    discount_policy: DiscountPolicy

    def total(self) -> Decimal:
        subtotal = sum(item.price for item in self.items)
        return self.discount_policy.apply(subtotal)
```
