# API Design: Python Examples

## Explicit Inputs: No Hidden Dependencies

```python
# Bad: hidden dependency on global state
def get_recommendations() -> list[Product]:
    user = current_session.user         # hidden
    region = os.environ['REGION']       # hidden
    return recommendation_engine.get(user, region)

# Good: all inputs declared
def get_recommendations(
    user_id: str,
    region: str,
    engine: RecommendationEngine
) -> list[Product]:
    return engine.get(user_id, region)
```

## Minimal Parameter Surface

```python
# Bad: caller must know internal field names
def create_order(
    customer_id: str,
    line_item_skus: list[str],
    line_item_qtys: list[int],
    line_item_prices: list[Decimal],
    shipping_address_line1: str,
    shipping_city: str
) -> Order: ...

# Good: group related inputs into a dataclass
@dataclass
class CreateOrderRequest:
    customer_id: str
    items: list[LineItemRequest]
    shipping_address: Address

def create_order(request: CreateOrderRequest) -> Order: ...
```

## Contract-First Documentation

```python
def place_order(request: CreateOrderRequest) -> Order:
    """
    Places a new order for the given customer.

    Raises InvalidOrderError if items is empty.
    Raises CustomerNotFoundError if customer_id does not exist.
    Returns the created Order with a server-assigned id.
    """
```

## Stable Versioning

```python
# Add optional fields: backwards compatible
@dataclass
class CreateOrderRequest:
    customer_id: str
    items: list[LineItemRequest]
    shipping_address: Address
    promo_code: str | None = None    # added later: optional, safe
```
