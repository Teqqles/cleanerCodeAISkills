# Design Patterns: Python Examples

## Strategy

```python
from abc import ABC, abstractmethod

class PricingStrategy(ABC):
    @abstractmethod
    def calculate_price(self, base_price: float, quantity: int) -> float: ...

class StandardPricing(PricingStrategy):
    def calculate_price(self, base_price: float, quantity: int) -> float:
        return base_price * quantity

class BulkPricing(PricingStrategy):
    def calculate_price(self, base_price: float, quantity: int) -> float:
        if quantity > 100:
            discount = 0.8
        elif quantity > 50:
            discount = 0.9
        else:
            discount = 1.0
        return base_price * quantity * discount

class OrderCalculator:
    def __init__(self, pricing: PricingStrategy):
        self._pricing = pricing

    def total(self, base_price: float, quantity: int) -> float:
        return self._pricing.calculate_price(base_price, quantity)
```

Using `Protocol` for structural typing (no inheritance required):

```python
from typing import Protocol

class PricingStrategy(Protocol):
    def calculate_price(self, base_price: float, quantity: int) -> float: ...
```

## Observer

```python
from typing import Callable

class EventEmitter[T]:
    def __init__(self):
        self._listeners: list[Callable[[T], None]] = []

    def subscribe(self, listener: Callable[[T], None]) -> Callable[[], None]:
        self._listeners.append(listener)
        return lambda: self._listeners.remove(listener)

    def emit(self, event: T) -> None:
        for listener in self._listeners:
            listener(event)

class OrderService:
    def __init__(self):
        self.order_placed = EventEmitter[Order]()

    def place(self, cart: Cart) -> Order:
        order = Order.create(cart)
        self.order_placed.emit(order)
        return order
```

## Factory

```python
def create_notification_channel(channel_type: str) -> NotificationChannel:
    channels = {
        'email': EmailChannel,
        'sms': SmsChannel,
        'push': PushChannel,
    }
    if channel_type not in channels:
        raise ValueError(f"Unknown channel: {channel_type}")
    return channels[channel_type]()
```

## Decorator

```python
class LoggingHttpClient:
    def __init__(self, inner: HttpClient, logger: Logger):
        self._inner = inner
        self._logger = logger

    def get(self, url: str) -> Response:
        self._logger.info(f"GET {url}")
        response = self._inner.get(url)
        self._logger.info(f"{url} -> {response.status}")
        return response

class RetryingHttpClient:
    def __init__(self, inner: HttpClient, max_retries: int):
        self._inner = inner
        self._max_retries = max_retries

    def get(self, url: str) -> Response:
        last_error: Exception | None = None
        for _ in range(self._max_retries + 1):
            try:
                return self._inner.get(url)
            except Exception as e:
                last_error = e
        raise last_error

# Compose
client = RetryingHttpClient(
    LoggingHttpClient(FetchHttpClient(), logger),
    max_retries=3
)
```

## Adapter

```python
class StripeAdapter:
    def __init__(self, stripe_client: StripeClient):
        self._stripe = stripe_client

    def charge(self, amount: float, currency: str) -> PaymentResult:
        intent = self._stripe.payment_intents.create(
            amount=round(amount * 100),
            currency=currency,
        )
        status = 'ok' if intent.status == 'succeeded' else 'failed'
        return PaymentResult(id=intent.id, status=status)
```
