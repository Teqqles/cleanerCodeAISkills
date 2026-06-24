# Single Responsibility: Python Examples

## Splitting a Multi-Responsibility Class

```python
# Violated: UserService handles persistence, validation, and notifications
class UserService:
    def __init__(self, db: Database, mailer: Mailer):
        self._db = db
        self._mailer = mailer

    def register(self, data: UserInput) -> User:
        if "@" not in data.email:
            raise InvalidInputError("bad email")
        user = User(email=data.email, name=data.name)
        self._db.save(user)
        self._mailer.send(user.email, "Welcome!")
        return user

# Fixed: each class has one reason to change
class UserValidator:
    def validate(self, data: UserInput) -> None:
        if "@" not in data.email:
            raise InvalidInputError("bad email")

class UserRepository:
    def __init__(self, db: Database):
        self._db = db

    def save(self, user: User) -> None:
        self._db.save(user)

class WelcomeNotifier:
    def __init__(self, mailer: Mailer):
        self._mailer = mailer

    def notify(self, user: User) -> None:
        self._mailer.send(user.email, "Welcome!")

class RegisterUser:
    def __init__(self, validator: UserValidator, repo: UserRepository, notifier: WelcomeNotifier):
        self._validator = validator
        self._repo = repo
        self._notifier = notifier

    def execute(self, data: UserInput) -> User:
        self._validator.validate(data)
        user = User(email=data.email, name=data.name)
        self._repo.save(user)
        self._notifier.notify(user)
        return user
```

## Function-Level SRP: Separate Orchestration from Detail

```python
# Violated: mixes calculation with formatting
def generate_report(orders: list[Order]) -> str:
    total = sum(o.amount for o in orders)
    lines = [f"{o.id}: {o.amount:.2f}" for o in orders]
    return f"Report\n" + "\n".join(lines) + f"\nTotal: {total:.2f}"

# Fixed: each function has one job
def calculate_total(orders: list[Order]) -> float:
    return sum(o.amount for o in orders)

def format_report(orders: list[Order], total: float) -> str:
    lines = [f"{o.id}: {o.amount:.2f}" for o in orders]
    return f"Report\n" + "\n".join(lines) + f"\nTotal: {total:.2f}"

def generate_report(orders: list[Order]) -> str:
    total = calculate_total(orders)
    return format_report(orders, total)
```

## Module-Level SRP

```
# Violated: order package handles payments too
src/order/
├── order.py
├── place_order.py
├── charge_card.py       # payment concern
└── refund_payment.py    # payment concern

# Fixed: separate packages by axis of change
src/order/
├── order.py
└── place_order.py

src/payment/
├── charge_card.py
└── refund_payment.py
```
