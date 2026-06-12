# Engineering Rigor: Python Examples

## Typed Errors: No Silent Failures

```python
# Bad: error swallowed
async def charge_customer(order: Order) -> None:
    try:
        await payment_gateway.charge(order)
    except Exception:
        pass  # silently ignored

# Good: typed result, failure surfaced
from dataclasses import dataclass

@dataclass
class ChargeSuccess:
    charge_id: str

@dataclass
class ChargeFailure:
    reason: str

async def charge_customer(order: Order) -> ChargeSuccess | ChargeFailure:
    try:
        charge_id = await payment_gateway.charge(order)
        return ChargeSuccess(charge_id=charge_id)
    except PaymentGatewayError as err:
        logger.error('charge_failed', order_id=order.id, error=str(err))
        return ChargeFailure(reason='gateway_error')
```

## Explicit Precondition Enforcement

```python
# Bad: crashes with unhelpful message later
def apply_discount(price: Decimal, rate: Decimal) -> Decimal:
    return price - price * rate

# Good: preconditions checked at the boundary
def apply_discount(price: Decimal, rate: Decimal) -> Decimal:
    if price < 0:
        raise InvalidPriceError(f'price must be non-negative, got {price}')
    if not (0 <= rate <= 1):
        raise InvalidDiscountError(f'rate must be 0-1, got {rate}')
    return price - price * rate
```

## Structured Logging

```python
# Bad: unstructured, hard to query
print(f'order placed for user {user_id}')

# Good: structured, filterable
logger.info('order_placed', extra={
    'order_id': order.id,
    'customer_id': order.customer_id,
    'total_amount': str(order.total()),
    'item_count': len(order.items)
})
```

## Environment-Driven Configuration

```python
# Bad: hardcoded
DB_URL = 'postgres://localhost:5432/mydb'

# Good: from environment
DB_URL = os.environ.get('DATABASE_URL')
if not DB_URL:
    raise ConfigurationError('DATABASE_URL is required')
```
