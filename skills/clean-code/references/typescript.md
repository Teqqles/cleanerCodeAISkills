# Clean Code: TypeScript Examples

## Naming

```typescript
// Bad
const d = new Date()
const arr = users.filter(u => u.a > 18)

// Good
const currentDate = new Date()
const adultUsers = users.filter(user => user.age > 18)
```

## Early Returns

```typescript
// Bad
function getDiscount(user: User): number {
  if (user) {
    if (user.isPremium) {
      if (user.yearsActive > 2) {
        return 0.2
      } else {
        return 0.1
      }
    }
  }
  return 0
}

// Good
function getDiscount(user: User): number {
  if (!user) return 0
  if (!user.isPremium) return 0
  return user.yearsActive > 2 ? 0.2 : 0.1
}
```

## Single-Purpose Functions

```typescript
// Bad: validates, saves, and sends email in one function
function processUser(user: User): void {
  if (!user.email) throw new Error('missing email')
  db.save(user)
  emailService.sendWelcome(user.email)
}

// Good: each function does one thing
function validateUser(user: User): void {
  if (!user.email) throw new InvalidUserError('email is required')
}

function registerUser(user: User, db: UserRepository, email: EmailService): void {
  validateUser(user)
  db.save(user)
  email.sendWelcome(user.email)
}
```

## Domain-Centric Structure

```
src/
├── order/
│   ├── Order.ts
│   ├── PlaceOrder.ts
│   └── order.test.ts
├── payment/
│   ├── Payment.ts
│   └── processPayment.ts
└── user/
    ├── User.ts
    └── registerUser.ts
```
