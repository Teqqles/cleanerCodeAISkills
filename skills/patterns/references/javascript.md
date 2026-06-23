# Design Patterns: JavaScript Examples

## Strategy

```javascript
/** @typedef {(basePrice: number, quantity: number) => number} PricingStrategy */

/** @type {PricingStrategy} */
const standardPricing = (basePrice, quantity) => basePrice * quantity

/** @type {PricingStrategy} */
const bulkPricing = (basePrice, quantity) => {
  const discount = quantity > 100 ? 0.8 : quantity > 50 ? 0.9 : 1.0
  return basePrice * quantity * discount
}

class OrderCalculator {
  constructor(pricing) {
    this.pricing = pricing
  }

  total(basePrice, quantity) {
    return this.pricing(basePrice, quantity)
  }
}
```

## Observer

```javascript
function createEventEmitter() {
  const listeners = []

  return {
    subscribe(listener) {
      listeners.push(listener)
      return () => {
        const index = listeners.indexOf(listener)
        if (index >= 0) listeners.splice(index, 1)
      }
    },
    emit(event) {
      listeners.forEach(listener => listener(event))
    }
  }
}

class OrderService {
  constructor() {
    this.orderPlaced = createEventEmitter()
  }

  place(cart) {
    const order = Order.create(cart)
    this.orderPlaced.emit(order)
    return order
  }
}
```

## Factory

```javascript
function createNotificationChannel(type) {
  const channels = {
    email: () => new EmailChannel(),
    sms: () => new SmsChannel(),
    push: () => new PushChannel(),
  }

  const create = channels[type]
  if (!create) throw new Error(`Unknown channel: ${type}`)
  return create()
}
```

## Decorator

```javascript
class LoggingHttpClient {
  constructor(inner, logger) {
    this.inner = inner
    this.logger = logger
  }

  async get(url) {
    this.logger.info(`GET ${url}`)
    const response = await this.inner.get(url)
    this.logger.info(`${url} -> ${response.status}`)
    return response
  }
}

class RetryingHttpClient {
  constructor(inner, maxRetries) {
    this.inner = inner
    this.maxRetries = maxRetries
  }

  async get(url) {
    let lastError
    for (let i = 0; i <= this.maxRetries; i++) {
      try { return await this.inner.get(url) }
      catch (e) { lastError = e }
    }
    throw lastError
  }
}

// Compose
const client = new RetryingHttpClient(
  new LoggingHttpClient(new FetchHttpClient(), logger),
  3
)
```

## Adapter

```javascript
class StripeAdapter {
  constructor(stripeClient) {
    this.stripe = stripeClient
  }

  async charge(amount, currency) {
    const intent = await this.stripe.paymentIntents.create({
      amount: Math.round(amount * 100),
      currency,
    })
    return {
      id: intent.id,
      status: intent.status === 'succeeded' ? 'ok' : 'failed',
    }
  }
}
```
