# Quality Testing: Python Examples

Test framework: **pytest**

## R: Right (correct output)

```python
def test_returns_order_total_from_line_items():
    order = Order([LineItem('book', Decimal('10')), LineItem('pen', Decimal('2'))])
    assert order.total() == Decimal('12')
```

## I: Inverse (roundtrip)

```python
def test_serialising_then_deserialising_returns_original():
    order = Order.create(customer_id='c1', items=[LineItem('book', Decimal('10'))])
    restored = Order.from_dict(order.to_dict())
    assert restored == order
```

## H: Harmful (invalid inputs)

```python
def test_raises_when_line_item_price_is_negative():
    with pytest.raises(InvalidPriceError):
        LineItem('book', Decimal('-1'))

def test_raises_when_item_name_is_empty():
    with pytest.raises(InvalidItemError):
        LineItem('', Decimal('10'))
```

## T: Time (fake clock)

```python
def test_marks_order_as_expired_after_30_minutes():
    clock = FakeClock(datetime(2024, 1, 1, 9, 0, 0))
    order = Order.create(cart, clock)
    clock.advance(minutes=31)
    assert order.is_expired(clock.now()) is True
```

## B: Boundaries

```python
def test_returns_zero_total_for_empty_order():
    assert Order([]).total() == Decimal('0')

def test_handles_order_with_one_item():
    assert Order([LineItem('book', Decimal('10'))]).total() == Decimal('10')
```

## E: Error Conditions

```python
def test_raises_order_not_found_when_id_missing():
    repo = InMemoryOrderRepository()
    with pytest.raises(OrderNotFoundError):
        repo.find_by_id('missing')
```
