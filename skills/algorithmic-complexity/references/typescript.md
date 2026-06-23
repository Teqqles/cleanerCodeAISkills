# Algorithmic Complexity: TypeScript Examples

## Hidden Quadratic in Array Methods

```typescript
function removeDuplicates(items: string[]): string[] {
  return items.filter((item, index) => items.indexOf(item) === index)
}
```

**Complexity:** Time O(n^2) where n is items.length. Space O(n) for the output.

**What this means:** `indexOf` scans up to n elements, called n times. For 10K items this is up to 100M comparisons.

**Can it be improved?** Yes, equally readable:

```typescript
function removeDuplicates(items: string[]): string[] {
  return [...new Set(items)]
}
```

**Complexity:** Time O(n), Space O(n). Set insertion and iteration are both linear. Strictly better with no readability cost.

---

## Nested Lookup Replaced by Map

```typescript
function findMatchingOrders(orders: Order[], customerIds: string[]): Order[] {
  return orders.filter(order => customerIds.includes(order.customerId))
}
```

**Complexity:** Time O(n * m) where n = orders, m = customerIds. Space O(n) worst case for output.

**Can it be improved?**

```typescript
function findMatchingOrders(orders: Order[], customerIds: string[]): Order[] {
  const idSet = new Set(customerIds)
  return orders.filter(order => idSet.has(order.customerId))
}
```

**Complexity:** Time O(n + m), Space O(m) for the Set plus O(n) for output. Equally readable, strictly faster.

---

## Sorting with a Custom Comparator

```typescript
function topNByRevenue(customers: Customer[], n: number): Customer[] {
  return [...customers]
    .sort((a, b) => b.revenue - a.revenue)
    .slice(0, n)
}
```

**Complexity:** Time O(k log k) where k = customers.length. Space O(k) for the copy.

**What this means:** Sorts all customers even though only the top n are needed. For k=1M and n=10, this does ~20M operations.

**Can it be improved?** A partial sort or min-heap gives O(k log n) time. However, the sort+slice version is vastly more readable and correct by inspection. Only switch to a heap if profiling shows this is a bottleneck with large k and small n.

---

## Map Construction

```typescript
function indexByEmail(users: User[]): Map<string, User> {
  const map = new Map<string, User>()
  for (const user of users) {
    map.set(user.email, user)
  }
  return map
}
```

**Complexity:** Time O(n), Space O(n) where n = users.length.

**What this means:** Each insertion is O(1) amortised. The map stores one entry per user. This is optimal for building a lookup structure.

**Can it be improved?** No. This is the lower bound for constructing an index. Every element must be visited at least once.
