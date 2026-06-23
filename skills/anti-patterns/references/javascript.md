# Anti-Patterns and Code Smells: JavaScript Examples

## God Class

```javascript
// Smell: one class doing unrelated work
class OrderManager {
  placeOrder(cart) { /* ... */ }
  cancelOrder(id) { /* ... */ }
  checkInventory(sku) { /* ... */ }
  reserveStock(sku, qty) { /* ... */ }
  sendConfirmationEmail(order) { /* ... */ }
  generateMonthlyReport() { /* ... */ }
}

// Fix: focused modules
class OrderService {
  placeOrder(cart) { /* ... */ }
  cancelOrder(id) { /* ... */ }
}

class InventoryService {
  checkStock(sku) { /* ... */ }
  reserve(sku, qty) { /* ... */ }
}

class OrderNotifications {
  sendConfirmation(order) { /* ... */ }
}
```

## Feature Envy

```javascript
// Smell: function reaching into another object's structure
function formatInvoiceAddress(order) {
  const c = order.getCustomer()
  const a = c.getAddress()
  return `${c.getFirstName()} ${c.getLastName()}\n${a.getStreet()}\n${a.getCity()}, ${a.getPostcode()}`
}

// Fix: ask the object to do the work
class Customer {
  formatAddress() {
    return `${this.firstName} ${this.lastName}\n${this.address.street}\n${this.address.city}, ${this.address.postcode}`
  }
}
```

## Primitive Obsession

```javascript
// Smell: bare strings, validation duplicated at every call site
function createUser(email, phone, role) {
  if (!email.includes('@')) throw new Error('invalid email')
  return { email, phone, role }
}

// Fix: value objects with validation at construction
class EmailAddress {
  constructor(raw) {
    if (!raw.includes('@')) throw new InvalidEmailError(raw)
    this.value = raw.toLowerCase()
  }
}

class PhoneNumber {
  constructor(raw) {
    const digits = raw.replace(/\D/g, '')
    if (digits.length < 10) throw new InvalidPhoneError(raw)
    this.value = digits
  }
}

function createUser(email, phone, role) {
  return { email, phone, role }
}
// Callers: createUser(new EmailAddress(raw), new PhoneNumber(raw), role)
```

## Flag Arguments

```javascript
// Smell: boolean selects between two behaviours
function sendNotification(user, urgent) {
  if (urgent) {
    sms.send(user.phone, buildUrgentMessage(user))
  } else {
    email.send(user.email, buildStandardMessage(user))
  }
}

// Fix: named functions
function sendUrgentNotification(user) {
  sms.send(user.phone, buildUrgentMessage(user))
}

function sendStandardNotification(user) {
  email.send(user.email, buildStandardMessage(user))
}
```

## Temporal Coupling

```javascript
// Smell: methods must be called in order
class ReportGenerator {
  init(config) { /* ... */ }
  loadData() { /* ... */ }
  process() { /* ... */ }
}

// Fix: single entry point enforces the sequence
class ReportGenerator {
  static generate(config) {
    const data = loadData(config)
    return processData(data)
  }
}
```

## Data Clumps

```javascript
// Smell: lat/lon always passed together
function calculateDistance(lat1, lon1, lat2, lon2) { /* ... */ }
function formatLocation(lat, lon) { /* ... */ }

// Fix: named object
function calculateDistance(from, to) { /* ... */ }
function formatLocation(coord) { /* ... */ }
// Callers: calculateDistance({ latitude: 51.5, longitude: -0.1 }, { latitude: 48.8, longitude: 2.3 })
```

## Inappropriate Intimacy via Module Globals

```javascript
// Smell: modules coupled through shared mutable state
// config.js
export let currentUser = null
export let apiBase = ''

// orderService.js
import { currentUser, apiBase } from './config.js'
function placeOrder(cart) {
  fetch(`${apiBase}/orders`, { headers: { user: currentUser.id } })
}

// Fix: pass dependencies explicitly
function placeOrder(cart, apiBase, currentUser) {
  fetch(`${apiBase}/orders`, { headers: { user: currentUser.id } })
}
```
