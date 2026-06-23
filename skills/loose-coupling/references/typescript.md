# Loose Coupling: TypeScript Examples

## Depend on Abstractions

```typescript
// Coupled: imports a concrete class from another module
import { PostgresUserStore } from '../infrastructure/PostgresUserStore'

class UserService {
  private store = new PostgresUserStore()
}

// Decoupled: depends on an interface the consumer owns
interface UserStore {
  find(id: string): Promise<User | null>
  save(user: User): Promise<void>
}

class UserService {
  constructor(private store: UserStore) {}
}
```

## Law of Demeter

```typescript
// Coupled: traverses the object graph
function getShippingCity(order: Order): string {
  return order.getCustomer().getAddress().getCity()
}

// Decoupled: ask the object directly
function getShippingCity(order: Order): string {
  return order.shippingCity()
}
```

## Tell, Don't Ask

```typescript
// Ask: caller makes decisions based on object's state
function maybeExpire(session: Session): void {
  if (Date.now() - session.getLastActivity() > session.getTimeout()) {
    session.setExpired(true)
    notifyUser(session.getUserId())
  }
}

// Tell: object owns the decision
class Session {
  checkExpiry(onExpired: (userId: string) => void): void {
    if (this.isExpired()) {
      onExpired(this.userId)
    }
  }
}
```

## Interface Segregation

```typescript
// Broad: forces consumers to depend on methods they don't use
interface DataStore {
  read(key: string): Promise<string>
  write(key: string, value: string): Promise<void>
  delete(key: string): Promise<void>
  list(): Promise<string[]>
  watch(key: string, cb: (value: string) => void): void
}

// Segregated: each consumer depends only on what it needs
interface Readable {
  read(key: string): Promise<string>
}

interface Writable {
  write(key: string, value: string): Promise<void>
}

interface Watchable {
  watch(key: string, cb: (value: string) => void): void
}
```

## Events Over Direct Calls

```typescript
// Coupled: OrderService knows about inventory, email, and analytics
class OrderService {
  constructor(
    private inventory: InventoryService,
    private email: EmailService,
    private analytics: AnalyticsService,
  ) {}

  place(cart: Cart): Order {
    const order = Order.create(cart)
    this.inventory.reserve(order.items)
    this.email.sendConfirmation(order)
    this.analytics.track('order_placed', order)
    return order
  }
}

// Decoupled: OrderService emits an event, consumers subscribe independently
class OrderService {
  readonly orderPlaced = new EventEmitter<Order>()

  place(cart: Cart): Order {
    const order = Order.create(cart)
    this.orderPlaced.emit(order)
    return order
  }
}

// Wired at the composition root
orderService.orderPlaced.subscribe(order => inventory.reserve(order.items))
orderService.orderPlaced.subscribe(order => email.sendConfirmation(order))
orderService.orderPlaced.subscribe(order => analytics.track('order_placed', order))
```

## Extension Without Modification

```typescript
// Coupled: every new export format edits this function
function exportReport(report: Report, format: string): string {
  if (format === 'csv') return toCsv(report)
  if (format === 'json') return toJson(report)
  if (format === 'xml') return toXml(report)
  throw new Error(`Unknown format: ${format}`)
}

// Decoupled: new formats are new implementations
interface ReportExporter {
  export(report: Report): string
}

class CsvExporter implements ReportExporter {
  export(report: Report): string { return toCsv(report) }
}

class JsonExporter implements ReportExporter {
  export(report: Report): string { return toJson(report) }
}

// Adding XML means adding XmlExporter, not editing existing code
```
