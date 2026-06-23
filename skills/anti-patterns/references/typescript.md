# Anti-Patterns and Code Smells: TypeScript Examples

## God Class

```typescript
// Smell: OrderManager handles orders, inventory, notifications, and reporting
class OrderManager {
  placeOrder(cart: Cart): Order { /* ... */ }
  cancelOrder(id: string): void { /* ... */ }
  checkInventory(sku: string): number { /* ... */ }
  reserveStock(sku: string, qty: number): void { /* ... */ }
  sendConfirmationEmail(order: Order): void { /* ... */ }
  generateMonthlyReport(): Report { /* ... */ }
}

// Fix: separate responsibilities into focused classes
class OrderService {
  placeOrder(cart: Cart): Order { /* ... */ }
  cancelOrder(id: string): void { /* ... */ }
}

class InventoryService {
  checkStock(sku: string): number { /* ... */ }
  reserve(sku: string, qty: number): void { /* ... */ }
}

class OrderNotifications {
  sendConfirmation(order: Order): void { /* ... */ }
}
```

## Feature Envy

```typescript
// Smell: reaches deep into customer's internals
function formatInvoiceAddress(order: Order): string {
  const customer = order.getCustomer()
  return `${customer.getFirstName()} ${customer.getLastName()}\n` +
    `${customer.getAddress().getStreet()}\n` +
    `${customer.getAddress().getCity()}, ${customer.getAddress().getPostcode()}`
}

// Fix: the behaviour belongs on the object that owns the data
class Customer {
  formatAddress(): string {
    return `${this.firstName} ${this.lastName}\n` +
      `${this.address.street}\n` +
      `${this.address.city}, ${this.address.postcode}`
  }
}
```

## Primitive Obsession

```typescript
// Smell: string everywhere, no validation boundary
function createUser(email: string, phone: string, role: string): User {
  // validation repeated at every call site
  if (!email.includes('@')) throw new Error('invalid email')
  return { email, phone, role }
}

// Fix: domain types that validate at construction
class EmailAddress {
  readonly value: string
  constructor(raw: string) {
    if (!raw.includes('@')) throw new InvalidEmailError(raw)
    this.value = raw.toLowerCase()
  }
}

class PhoneNumber {
  readonly value: string
  constructor(raw: string) {
    const digits = raw.replace(/\D/g, '')
    if (digits.length < 10) throw new InvalidPhoneError(raw)
    this.value = digits
  }
}

function createUser(email: EmailAddress, phone: PhoneNumber, role: UserRole): User {
  return { email, phone, role }
}
```

## Flag Arguments

```typescript
// Smell: boolean selects between two behaviours
function sendNotification(user: User, urgent: boolean): void {
  if (urgent) {
    sms.send(user.phone, buildUrgentMessage(user))
  } else {
    email.send(user.email, buildStandardMessage(user))
  }
}

// Fix: two named functions
function sendUrgentNotification(user: User): void {
  sms.send(user.phone, buildUrgentMessage(user))
}

function sendStandardNotification(user: User): void {
  email.send(user.email, buildStandardMessage(user))
}
```

## Temporal Coupling

```typescript
// Smell: must call init() before process(), nothing enforces it
class ReportGenerator {
  init(config: Config): void { /* ... */ }
  loadData(): void { /* ... */ }
  process(): Report { /* ... */ }
}

// Fix: make the sequence impossible to violate
class ReportGenerator {
  static generate(config: Config): Report {
    const data = loadData(config)
    return processData(data)
  }
}
```

## Data Clumps

```typescript
// Smell: same parameters travel together everywhere
function calculateDistance(lat1: number, lon1: number, lat2: number, lon2: number): number { /* ... */ }
function formatLocation(lat: number, lon: number): string { /* ... */ }
function isWithinRadius(lat: number, lon: number, centerLat: number, centerLon: number, radius: number): boolean { /* ... */ }

// Fix: extract a type that names the concept
interface Coordinate {
  latitude: number
  longitude: number
}

function calculateDistance(from: Coordinate, to: Coordinate): number { /* ... */ }
function formatLocation(coord: Coordinate): string { /* ... */ }
function isWithinRadius(point: Coordinate, center: Coordinate, radius: number): boolean { /* ... */ }
```
