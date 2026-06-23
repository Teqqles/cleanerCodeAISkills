# Design Patterns: Scala Examples

## Strategy

```scala
trait PricingStrategy {
  def calculatePrice(basePrice: BigDecimal, quantity: Int): BigDecimal
}

class StandardPricing extends PricingStrategy {
  def calculatePrice(basePrice: BigDecimal, quantity: Int): BigDecimal =
    basePrice * quantity
}

class BulkPricing extends PricingStrategy {
  def calculatePrice(basePrice: BigDecimal, quantity: Int): BigDecimal = {
    val discount = if (quantity > 100) 0.8 else if (quantity > 50) 0.9 else 1.0
    basePrice * quantity * discount
  }
}

class OrderCalculator(pricing: PricingStrategy) {
  def total(basePrice: BigDecimal, quantity: Int): BigDecimal =
    pricing.calculatePrice(basePrice, quantity)
}
```

Scala idiom: strategies as functions when the interface is a single method:

```scala
type PricingStrategy = (BigDecimal, Int) => BigDecimal

val bulkPricing: PricingStrategy = (base, qty) => {
  val discount = if (qty > 100) 0.8 else if (qty > 50) 0.9 else 1.0
  base * qty * discount
}

class OrderCalculator(pricing: PricingStrategy) {
  def total(base: BigDecimal, qty: Int): BigDecimal = pricing(base, qty)
}
```

## Observer

```scala
class EventEmitter[T] {
  private var listeners: List[T => Unit] = Nil

  def subscribe(listener: T => Unit): () => Unit = {
    listeners = listener :: listeners
    () => { listeners = listeners.filterNot(_ eq listener) }
  }

  def emit(event: T): Unit =
    listeners.foreach(_(event))
}

class OrderService {
  val orderPlaced = new EventEmitter[Order]

  def place(cart: Cart): Order = {
    val order = Order.create(cart)
    orderPlaced.emit(order)
    order
  }
}
```

## Factory

```scala
trait NotificationChannel {
  def send(message: String, recipient: String): Unit
}

object NotificationChannel {
  def create(channelType: String): NotificationChannel = channelType match {
    case "email" => new EmailChannel
    case "sms"   => new SmsChannel
    case "push"  => new PushChannel
    case other   => throw new IllegalArgumentException(s"Unknown channel: $other")
  }
}
```

## Decorator

```scala
trait HttpClient {
  def get(url: String): Response
}

class LoggingHttpClient(inner: HttpClient, logger: Logger) extends HttpClient {
  def get(url: String): Response = {
    logger.info(s"GET $url")
    val response = inner.get(url)
    logger.info(s"$url -> ${response.status}")
    response
  }
}

class RetryingHttpClient(inner: HttpClient, maxRetries: Int) extends HttpClient {
  def get(url: String): Response = {
    var lastError: Exception = null
    for (_ <- 0 to maxRetries) {
      try return inner.get(url)
      catch { case e: Exception => lastError = e }
    }
    throw lastError
  }
}

// Compose
val client: HttpClient = new RetryingHttpClient(
  new LoggingHttpClient(new OkHttpClient, logger),
  maxRetries = 3
)
```

## Adapter

```scala
class StripeAdapter(stripe: StripeClient) extends PaymentGateway {
  def charge(amount: BigDecimal, currency: String): PaymentResult = {
    val intent = stripe.paymentIntents.create(
      (amount * 100).toInt,
      currency
    )
    val status = if (intent.status == "succeeded") "ok" else "failed"
    PaymentResult(id = intent.id, status = status)
  }
}
```
