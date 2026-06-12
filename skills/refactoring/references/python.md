# Refactoring: Python Examples

## Extract Condition Into Named Function

```python
# Before
if (user.subscription_end_date > datetime.now()
        and user.plan != 'free'
        and not user.suspended):
    show_premium_content()

# After
def has_active_paid_subscription(user: User) -> bool:
    return (user.subscription_end_date > datetime.now()
            and user.plan != 'free'
            and not user.suspended)

if has_active_paid_subscription(user):
    show_premium_content()
```

## Replace Magic Values With Named Constants

```python
# Before
if response.status_code == 429:
    time.sleep(2)
    retry(3)

# After
HTTP_TOO_MANY_REQUESTS = 429
RETRY_DELAY_SECONDS = 2
MAX_RETRIES = 3

if response.status_code == HTTP_TOO_MANY_REQUESTS:
    time.sleep(RETRY_DELAY_SECONDS)
    retry(MAX_RETRIES)
```

## Delete Dead Code

```python
# Before
def calculate_shipping(order: Order) -> Decimal:
    # legacy_rate = Decimal('4.99')  # old flat rate, kept for reference
    weight = order.total_weight_kg()
    return weight * Decimal('1.5')

# After
def calculate_shipping(order: Order) -> Decimal:
    return order.total_weight_kg() * Decimal('1.5')
```

## Flatten Nesting With Early Returns

```python
# Before: 4 levels deep
def get_invoice_total(invoice: Invoice | None) -> Decimal:
    if invoice:
        if invoice.is_paid:
            if invoice.lines:
                return sum(line.amount for line in invoice.lines)
    return Decimal('0')

# After: flat
def get_invoice_total(invoice: Invoice | None) -> Decimal:
    if not invoice:
        return Decimal('0')
    if not invoice.is_paid:
        return Decimal('0')
    if not invoice.lines:
        return Decimal('0')
    return sum(line.amount for line in invoice.lines)
```
