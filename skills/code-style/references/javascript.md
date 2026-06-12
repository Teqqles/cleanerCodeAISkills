# Code Style: JavaScript

## Formatter

```bash
npx prettier --write .
npx eslint --fix .
```

Typical config files: `.prettierrc`, `eslint.config.js`

## Naming Conventions

```javascript
const userName = 'Alice'              // camelCase: variables
function calculateTotal() {}          // camelCase: functions
class OrderService {}                 // PascalCase: classes
const MAX_RETRIES = 3                 // UPPER_SNAKE_CASE: module-level constants
const _internalHelper = () => {}      // leading underscore: conventionally private
```

## Comments

```javascript
// Bad: describes what, not why
// Loop through users and add to total
const total = users.reduce((sum, user) => sum + user.balance, 0)

// Good: only when why is non-obvious
// Balances can be negative (credit accounts); sum without filtering is intentional
const total = users.reduce((sum, user) => sum + user.balance, 0)
```

## JSDoc: Contracts, Not Implementation

```javascript
// Bad
/** This method loops through orders and calculates the total price */
function calculateTotal(orders) { ... }

// Good
/**
 * @param {Order[]} orders
 * @returns {number} Sum of all order amounts. Returns 0 for an empty array.
 */
function calculateTotal(orders) { ... }
```
