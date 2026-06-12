# Code Style: TypeScript

## Formatter

```bash
npx prettier --write .
npx eslint --fix .
```

Typical config files: `.prettierrc`, `.eslintrc.json`

## Naming Conventions

```typescript
const userName = 'Alice'              // camelCase: variables
function calculateTotal() {}          // camelCase: functions
class OrderService {}                 // PascalCase: classes
interface UserRepository {}           // PascalCase: interfaces
type UserId = string                  // PascalCase: type aliases
enum PaymentMethod { Card, Transfer } // PascalCase: enums and members
const MAX_RETRIES = 3                 // UPPER_SNAKE_CASE: module-level constants
```

## Comments

```typescript
// Bad: describes what, not why
// Loop through users and add to total
let total = 0
for (const user of users) {
  total += user.balance
}

// Good: only when why is non-obvious
// Balances can be negative (credit accounts); sum without filtering is intentional
let total = users.reduce((sum, user) => sum + user.balance, 0)
```

## No Subjective Language in Documentation

```typescript
// Bad
/** Comprehensive utility for robust user management */
class UserService {}

// Good
/** Persists and retrieves users. Throws UserNotFoundError when lookup fails. */
class UserService {}
```
