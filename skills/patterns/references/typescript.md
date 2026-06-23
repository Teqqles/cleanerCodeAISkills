# Design Patterns: TypeScript Examples

## Strategy

```typescript
interface PricingStrategy {
  calculatePrice(basePrice: number, quantity: number): number
}

class StandardPricing implements PricingStrategy {
  calculatePrice(basePrice: number, quantity: number): number {
    return basePrice * quantity
  }
}

class BulkPricing implements PricingStrategy {
  calculatePrice(basePrice: number, quantity: number): number {
    const discount = quantity > 100 ? 0.8 : quantity > 50 ? 0.9 : 1.0
    return basePrice * quantity * discount
  }
}

class OrderCalculator {
  constructor(private pricing: PricingStrategy) {}

  total(basePrice: number, quantity: number): number {
    return this.pricing.calculatePrice(basePrice, quantity)
  }
}
```

## Observer

```typescript
type Listener<T> = (event: T) => void

class EventEmitter<T> {
  private listeners: Listener<T>[] = []

  subscribe(listener: Listener<T>): () => void {
    this.listeners.push(listener)
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener)
    }
  }

  emit(event: T): void {
    this.listeners.forEach(listener => listener(event))
  }
}

// Usage: producer does not know its consumers
class OrderService {
  readonly orderPlaced = new EventEmitter<Order>()

  place(cart: Cart): Order {
    const order = Order.create(cart)
    this.orderPlaced.emit(order)
    return order
  }
}
```

## Factory

```typescript
interface NotificationChannel {
  send(message: string, recipient: string): void
}

function createNotificationChannel(type: string): NotificationChannel {
  switch (type) {
    case 'email': return new EmailChannel()
    case 'sms': return new SmsChannel()
    case 'push': return new PushChannel()
    default: throw new Error(`Unknown channel: ${type}`)
  }
}
```

## Decorator

```typescript
interface HttpClient {
  get(url: string): Promise<Response>
}

class LoggingHttpClient implements HttpClient {
  constructor(private inner: HttpClient, private logger: Logger) {}

  async get(url: string): Promise<Response> {
    this.logger.info(`GET ${url}`)
    const response = await this.inner.get(url)
    this.logger.info(`${url} -> ${response.status}`)
    return response
  }
}

class RetryingHttpClient implements HttpClient {
  constructor(private inner: HttpClient, private maxRetries: number) {}

  async get(url: string): Promise<Response> {
    let lastError: Error
    for (let attempt = 0; attempt <= this.maxRetries; attempt++) {
      try { return await this.inner.get(url) }
      catch (e) { lastError = e as Error }
    }
    throw lastError!
  }
}

// Compose: retry wraps logging wraps the real client
const client = new RetryingHttpClient(
  new LoggingHttpClient(new FetchHttpClient(), logger),
  3
)
```

## Adapter

```typescript
interface PaymentGateway {
  charge(amount: number, currency: string): Promise<PaymentResult>
}

class StripeAdapter implements PaymentGateway {
  constructor(private stripe: StripeClient) {}

  async charge(amount: number, currency: string): Promise<PaymentResult> {
    const intent = await this.stripe.paymentIntents.create({
      amount: Math.round(amount * 100),
      currency,
    })
    return { id: intent.id, status: intent.status === 'succeeded' ? 'ok' : 'failed' }
  }
}
```
