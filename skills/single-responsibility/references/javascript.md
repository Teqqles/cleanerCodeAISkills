# Single Responsibility: JavaScript Examples

## Splitting a Multi-Responsibility Class

```javascript
// Violated: UserService handles persistence, validation, and notifications
class UserService {
  constructor(db, mailer) {
    this.db = db
    this.mailer = mailer
  }

  register(data) {
    if (!data.email.includes('@')) throw new Error('bad email')
    const user = { email: data.email, name: data.name }
    this.db.save(user)
    this.mailer.send(user.email, 'Welcome!')
    return user
  }
}

// Fixed: each class has one reason to change
class UserValidator {
  validate(data) {
    if (!data.email.includes('@')) throw new Error('bad email')
  }
}

class UserRepository {
  constructor(db) { this.db = db }
  save(user) { this.db.save(user) }
}

class WelcomeNotifier {
  constructor(mailer) { this.mailer = mailer }
  notify(user) { this.mailer.send(user.email, 'Welcome!') }
}

class RegisterUser {
  constructor(validator, repo, notifier) {
    this.validator = validator
    this.repo = repo
    this.notifier = notifier
  }

  execute(data) {
    this.validator.validate(data)
    const user = { email: data.email, name: data.name }
    this.repo.save(user)
    this.notifier.notify(user)
    return user
  }
}
```

## Function-Level SRP: Separate Orchestration from Detail

```javascript
// Violated: mixes calculation with formatting
function generateReport(orders) {
  const total = orders.reduce((sum, o) => sum + o.amount, 0)
  const lines = orders.map(o => `${o.id}: ${o.amount.toFixed(2)}`)
  return `Report\n${lines.join('\n')}\nTotal: ${total.toFixed(2)}`
}

// Fixed: each function does one thing
function calculateTotal(orders) {
  return orders.reduce((sum, o) => sum + o.amount, 0)
}

function formatReport(orders, total) {
  const lines = orders.map(o => `${o.id}: ${o.amount.toFixed(2)}`)
  return `Report\n${lines.join('\n')}\nTotal: ${total.toFixed(2)}`
}

function generateReport(orders) {
  const total = calculateTotal(orders)
  return formatReport(orders, total)
}
```

## Module-Level SRP

```
// Violated: order module includes payment logic
src/order/
├── order.js
├── placeOrder.js
├── chargeCard.js       // payment concern
└── refundPayment.js    // payment concern

// Fixed: separate modules by axis of change
src/order/
├── order.js
└── placeOrder.js

src/payment/
├── chargeCard.js
└── refundPayment.js
```
