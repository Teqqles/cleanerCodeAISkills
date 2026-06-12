# Clean Code: JavaScript Examples

## Naming

```javascript
// Bad
const d = new Date()
const arr = users.filter(u => u.a > 18)

// Good
const currentDate = new Date()
const adultUsers = users.filter(user => user.age > 18)
```

## Early Returns

```javascript
// Bad
function getDiscount(user) {
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
function getDiscount(user) {
  if (!user) return 0
  if (!user.isPremium) return 0
  return user.yearsActive > 2 ? 0.2 : 0.1
}
```

## Single-Purpose Functions

```javascript
// Bad: validates, saves, and sends email in one function
function processUser(user) {
  if (!user.email) throw new Error('missing email')
  db.save(user)
  emailService.sendWelcome(user.email)
}

// Good: each function does one thing
function validateUser(user) {
  if (!user.email) throw new InvalidUserError('email is required')
}

function registerUser(user, db, emailService) {
  validateUser(user)
  db.save(user)
  emailService.sendWelcome(user.email)
}
```

## Domain-Centric Structure

```
src/
├── order/
│   ├── Order.js
│   ├── placeOrder.js
│   └── order.test.js
├── payment/
│   ├── Payment.js
│   └── processPayment.js
└── user/
    ├── User.js
    └── registerUser.js
```
