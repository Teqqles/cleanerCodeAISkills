# Clean Code: Python Examples

## Naming

```python
# Bad
d = datetime.now()
arr = [u for u in users if u.a > 18]

# Good
current_date = datetime.now()
adult_users = [user for user in users if user.age > 18]
```

## Early Returns

```python
# Bad
def get_discount(user):
    if user:
        if user.is_premium:
            if user.years_active > 2:
                return Decimal('0.2')
            else:
                return Decimal('0.1')
    return Decimal('0')

# Good
def get_discount(user: User) -> Decimal:
    if not user:
        return Decimal('0')
    if not user.is_premium:
        return Decimal('0')
    return Decimal('0.2') if user.years_active > 2 else Decimal('0.1')
```

## Single-Purpose Functions

```python
# Bad: validates, saves, and sends email in one function
def process_user(user):
    if not user.email:
        raise ValueError('missing email')
    db.save(user)
    email_service.send_welcome(user.email)

# Good: each function does one thing
def validate_user(user: User) -> None:
    if not user.email:
        raise InvalidUserError('email is required')

def register_user(user: User, db: UserRepository, email: EmailService) -> None:
    validate_user(user)
    db.save(user)
    email.send_welcome(user.email)
```

## Domain-Centric Structure

```
src/
├── order/
│   ├── order.py
│   ├── place_order.py
│   └── test_order.py
├── payment/
│   ├── payment.py
│   └── process_payment.py
└── user/
    ├── user.py
    └── register_user.py
```
