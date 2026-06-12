# Test-Driven Development: Python Examples

Test framework: **pytest** (`pip install pytest`)

## Red: Write the Failing Test First

```python
# test_price_calculator.py
def test_applies_percentage_discount_to_base_price():
    calculator = PriceCalculator()
    result = calculator.apply_discount(Decimal('100'), Decimal('0.1'))
    assert result == Decimal('90.0')
# Run: pytest: test fails, PriceCalculator does not exist yet
```

## Green: Minimum Code to Pass

```python
# price_calculator.py
from decimal import Decimal

class PriceCalculator:
    def apply_discount(self, price: Decimal, discount_rate: Decimal) -> Decimal:
        return price - price * discount_rate
# Run: pytest: test passes
```

## Refactor: Improve Without Breaking

```python
class PriceCalculator:
    def apply_discount(self, price: Decimal, discount_rate: Decimal) -> Decimal:
        discount_amount = self._discount_amount_for(price, discount_rate)
        return price - discount_amount

    def _discount_amount_for(self, price: Decimal, rate: Decimal) -> Decimal:
        return price * rate
# Run: pytest: still passes
```

## Test Names Describe Behaviour

```python
def test_returns_full_price_when_discount_rate_is_zero():
    assert PriceCalculator().apply_discount(Decimal('100'), Decimal('0')) == Decimal('100')

def test_raises_when_discount_rate_exceeds_one():
    with pytest.raises(InvalidDiscountError):
        PriceCalculator().apply_discount(Decimal('100'), Decimal('1.5'))

def test_raises_when_price_is_negative():
    with pytest.raises(InvalidPriceError):
        PriceCalculator().apply_discount(Decimal('-1'), Decimal('0.1'))
```
