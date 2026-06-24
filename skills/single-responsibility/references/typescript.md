# Single Responsibility: TypeScript Examples

## Splitting a Multi-Responsibility Class

```typescript
// Violated: UserService handles persistence, validation, and notifications
class UserService {
  constructor(private db: Database, private mailer: Mailer) {}

  register(data: UserInput): User {
    if (!data.email.includes('@')) throw new InvalidInputError('bad email')
    const user = new User(data.email, data.name)
    this.db.save(user)
    this.mailer.send(user.email, 'Welcome!')
    return user
  }
}

// Fixed: each class has one reason to change
class UserValidator {
  validate(data: UserInput): void {
    if (!data.email.includes('@')) throw new InvalidInputError('bad email')
  }
}

class UserRepository {
  constructor(private db: Database) {}
  save(user: User): void { this.db.save(user) }
}

class WelcomeNotifier {
  constructor(private mailer: Mailer) {}
  notify(user: User): void { this.mailer.send(user.email, 'Welcome!') }
}

class RegisterUser {
  constructor(
    private validator: UserValidator,
    private repo: UserRepository,
    private notifier: WelcomeNotifier,
  ) {}

  execute(data: UserInput): User {
    this.validator.validate(data)
    const user = new User(data.email, data.name)
    this.repo.save(user)
    this.notifier.notify(user)
    return user
  }
}
```

## Function-Level SRP: Separate Orchestration from Detail

```typescript
// Violated: mixes orchestration and formatting detail
function generateReport(orders: Order[]): string {
  const total = orders.reduce((sum, o) => sum + o.amount, 0)
  const lines = orders.map(o => `${o.id}: ${o.amount.toFixed(2)}`)
  return `Report\n${lines.join('\n')}\nTotal: ${total.toFixed(2)}`
}

// Fixed: orchestration in one function, formatting in another
function calculateTotal(orders: Order[]): number {
  return orders.reduce((sum, o) => sum + o.amount, 0)
}

function formatReport(orders: Order[], total: number): string {
  const lines = orders.map(o => `${o.id}: ${o.amount.toFixed(2)}`)
  return `Report\n${lines.join('\n')}\nTotal: ${total.toFixed(2)}`
}

function generateReport(orders: Order[]): string {
  const total = calculateTotal(orders)
  return formatReport(orders, total)
}
```

## Module-Level SRP

```
// Violated: one module handles order logic and payment processing
src/order/
├── Order.ts
├── placeOrder.ts
├── chargeCard.ts        // payment concern mixed in
└── refundPayment.ts     // payment concern mixed in

// Fixed: each module has one axis of change
src/order/
├── Order.ts
└── placeOrder.ts

src/payment/
├── chargeCard.ts
└── refundPayment.ts
```
