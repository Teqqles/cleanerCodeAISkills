# Refactoring: JavaScript Examples

## Extract Condition Into Named Function

```javascript
// Before
if (user.subscriptionEndDate > new Date() && user.plan !== 'free' && !user.suspended) {
  showPremiumContent()
}

// After
function hasActivePaidSubscription(user) {
  return user.subscriptionEndDate > new Date()
    && user.plan !== 'free'
    && !user.suspended
}

if (hasActivePaidSubscription(user)) {
  showPremiumContent()
}
```

## Replace Magic Values With Named Constants

```javascript
// Before
if (response.status === 429) {
  await sleep(2000)
  retry(3)
}

// After
const HTTP_TOO_MANY_REQUESTS = 429
const RETRY_DELAY_MS = 2_000
const MAX_RETRIES = 3

if (response.status === HTTP_TOO_MANY_REQUESTS) {
  await sleep(RETRY_DELAY_MS)
  retry(MAX_RETRIES)
}
```

## Delete Dead Code

```javascript
// Before
function calculateShipping(order) {
  // const legacyRate = 4.99  // old flat rate, kept for reference
  const weight = order.totalWeightKg()
  return weight * 1.5
}

// After
function calculateShipping(order) {
  return order.totalWeightKg() * 1.5
}
```

## Flatten Nesting With Early Returns

```javascript
// Before: 4 levels deep
function getInvoiceTotal(invoice) {
  if (invoice) {
    if (invoice.isPaid) {
      if (invoice.lines.length > 0) {
        return invoice.lines.reduce((sum, l) => sum + l.amount, 0)
      }
    }
  }
  return 0
}

// After: flat
function getInvoiceTotal(invoice) {
  if (!invoice) return 0
  if (!invoice.isPaid) return 0
  if (invoice.lines.length === 0) return 0
  return invoice.lines.reduce((sum, l) => sum + l.amount, 0)
}
```
