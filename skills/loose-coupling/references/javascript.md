# Loose Coupling: JavaScript Examples

## Depend on Abstractions

```javascript
// Coupled: imports a concrete implementation
const { PostgresUserStore } = require('./infrastructure/PostgresUserStore')

class UserService {
  constructor() {
    this.store = new PostgresUserStore()
  }
}

// Decoupled: receives any object that satisfies the contract
class UserService {
  /** @param {{ find(id: string): User|null, save(user: User): void }} store */
  constructor(store) {
    this.store = store
  }
}
```

## Law of Demeter

```javascript
// Coupled: traverses the object graph
function getShippingCity(order) {
  return order.getCustomer().getAddress().getCity()
}

// Decoupled: ask the object
function getShippingCity(order) {
  return order.shippingCity()
}
```

## Tell, Don't Ask

```javascript
// Ask: caller interrogates then decides
function maybeExpire(session) {
  if (Date.now() - session.lastActivity > session.timeout) {
    session.expired = true
    notifyUser(session.userId)
  }
}

// Tell: session owns the decision
class Session {
  checkExpiry(onExpired) {
    if (this.isExpired()) {
      onExpired(this.userId)
    }
  }
}
```

## Interface Segregation (via focused objects)

```javascript
// Broad: one object expected to do everything
function processData(store) {
  store.read(key)
  store.write(key, value)
  store.delete(key)
  store.watch(key, cb)
}

// Segregated: accept only what you need
function readConfig(reader) {
  // reader only needs: { read(key): string }
  return reader.read('app.config')
}

function watchChanges(watcher) {
  // watcher only needs: { watch(key, cb): void }
  watcher.watch('app.config', handleChange)
}
```

## Events Over Direct Calls

```javascript
// Coupled: OrderService knows its consumers
class OrderService {
  constructor(inventory, email, analytics) {
    this.inventory = inventory
    this.email = email
    this.analytics = analytics
  }

  place(cart) {
    const order = Order.create(cart)
    this.inventory.reserve(order.items)
    this.email.sendConfirmation(order)
    this.analytics.track('order_placed', order)
    return order
  }
}

// Decoupled: emit events
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

// Composition root
orderService.orderPlaced.subscribe(order => inventory.reserve(order.items))
orderService.orderPlaced.subscribe(order => email.sendConfirmation(order))
orderService.orderPlaced.subscribe(order => analytics.track('order_placed', order))
```

## Extension Without Modification

```javascript
// Coupled: if chain grows with every format
function exportReport(report, format) {
  if (format === 'csv') return toCsv(report)
  if (format === 'json') return toJson(report)
  if (format === 'xml') return toXml(report)
  throw new Error(`Unknown format: ${format}`)
}

// Decoupled: registry of exporters
const exporters = {
  csv: report => toCsv(report),
  json: report => toJson(report),
}

function exportReport(report, format) {
  const exporter = exporters[format]
  if (!exporter) throw new Error(`Unknown format: ${format}`)
  return exporter(report)
}

// Adding XML: exporters.xml = report => toXml(report)
// No existing code edited
```

## No Shared Mutable State

```javascript
// Coupled: modules communicate through shared global
let sharedState = { user: null, token: null }

// module-a.js
sharedState.user = fetchUser()

// module-b.js
if (sharedState.user) doSomething(sharedState.user)

// Decoupled: explicit data flow
function moduleA(fetchUser) {
  return fetchUser()
}

function moduleB(user) {
  if (user) doSomething(user)
}

// Wired at the boundary
const user = moduleA(fetchUser)
moduleB(user)
```
