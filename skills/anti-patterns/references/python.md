# Anti-Patterns and Code Smells: Python Examples

## God Class

```python
# Smell: OrderManager does everything
class OrderManager:
    def place_order(self, cart: Cart) -> Order: ...
    def cancel_order(self, order_id: str) -> None: ...
    def check_inventory(self, sku: str) -> int: ...
    def reserve_stock(self, sku: str, qty: int) -> None: ...
    def send_confirmation_email(self, order: Order) -> None: ...
    def generate_monthly_report(self) -> Report: ...

# Fix: focused classes
class OrderService:
    def place_order(self, cart: Cart) -> Order: ...
    def cancel_order(self, order_id: str) -> None: ...

class InventoryService:
    def check_stock(self, sku: str) -> int: ...
    def reserve(self, sku: str, qty: int) -> None: ...

class OrderNotifications:
    def send_confirmation(self, order: Order) -> None: ...
```

## Feature Envy

```python
# Smell: reaches into customer internals
def format_invoice_address(order: Order) -> str:
    customer = order.get_customer()
    addr = customer.get_address()
    return f"{customer.first_name} {customer.last_name}\n{addr.street}\n{addr.city}, {addr.postcode}"

# Fix: behaviour belongs on the object that owns the data
class Customer:
    def format_address(self) -> str:
        return f"{self.first_name} {self.last_name}\n{self.address.street}\n{self.address.city}, {self.address.postcode}"
```

## Primitive Obsession

```python
# Smell: bare strings with repeated validation
def create_user(email: str, phone: str, role: str) -> User:
    if "@" not in email:
        raise ValueError("invalid email")
    return User(email=email, phone=phone, role=role)

# Fix: value types that validate at construction
@dataclass(frozen=True)
class EmailAddress:
    value: str

    def __post_init__(self):
        if "@" not in self.value:
            raise InvalidEmailError(self.value)
        object.__setattr__(self, "value", self.value.lower())

@dataclass(frozen=True)
class PhoneNumber:
    value: str

    def __post_init__(self):
        digits = "".join(c for c in self.value if c.isdigit())
        if len(digits) < 10:
            raise InvalidPhoneError(self.value)
        object.__setattr__(self, "value", digits)

def create_user(email: EmailAddress, phone: PhoneNumber, role: UserRole) -> User:
    return User(email=email, phone=phone, role=role)
```

## Flag Arguments

```python
# Smell: boolean selects between two behaviours
def send_notification(user: User, urgent: bool) -> None:
    if urgent:
        sms.send(user.phone, build_urgent_message(user))
    else:
        email.send(user.email, build_standard_message(user))

# Fix: two named functions
def send_urgent_notification(user: User) -> None:
    sms.send(user.phone, build_urgent_message(user))

def send_standard_notification(user: User) -> None:
    email.send(user.email, build_standard_message(user))
```

## Temporal Coupling

```python
# Smell: init() must be called before process()
class ReportGenerator:
    def init(self, config: Config) -> None: ...
    def load_data(self) -> None: ...
    def process(self) -> Report: ...

# Fix: enforce the sequence through the API
class ReportGenerator:
    @staticmethod
    def generate(config: Config) -> Report:
        data = load_data(config)
        return process_data(data)
```

## Data Clumps

```python
# Smell: lat/lon always travel together
def calculate_distance(lat1: float, lon1: float, lat2: float, lon2: float) -> float: ...
def format_location(lat: float, lon: float) -> str: ...

# Fix: named type
@dataclass(frozen=True)
class Coordinate:
    latitude: float
    longitude: float

def calculate_distance(start: Coordinate, end: Coordinate) -> float: ...
def format_location(coord: Coordinate) -> str: ...
```
