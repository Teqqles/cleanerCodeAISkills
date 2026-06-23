# Algorithmic Complexity: JavaScript Examples

## Hidden Quadratic in Array Methods

```javascript
function removeDuplicates(items) {
  return items.filter((item, index) => items.indexOf(item) === index)
}
```

**Complexity:** Time O(n^2) where n = items.length. Space O(n) for the output.

**What this means:** `indexOf` scans up to n elements, called n times. For 10K items, up to 100M comparisons.

**Can it be improved?** Yes, equally readable:

```javascript
function removeDuplicates(items) {
  return [...new Set(items)]
}
```

**Complexity:** Time O(n), Space O(n). Set provides O(1) insertion. Strictly better with no readability cost.

---

## Nested Lookup Replaced by Set

```javascript
function findMatchingOrders(orders, customerIds) {
  return orders.filter(order => customerIds.includes(order.customerId))
}
```

**Complexity:** Time O(n * m) where n = orders.length, m = customerIds.length. Space O(n) for output.

**Can it be improved?**

```javascript
function findMatchingOrders(orders, customerIds) {
  const idSet = new Set(customerIds)
  return orders.filter(order => idSet.has(order.customerId))
}
```

**Complexity:** Time O(n + m), Space O(m) for the Set. Equally readable, eliminates the quadratic.

---

## Object Key Construction

```javascript
function indexByEmail(users) {
  const index = {}
  for (const user of users) {
    index[user.email] = user
  }
  return index
}
```

**Complexity:** Time O(n), Space O(n) where n = users.length.

**What this means:** Property assignment on a plain object is O(1) amortised. Each user is visited once. This is optimal.

**Can it be improved?** No. Linear is the lower bound for building a lookup from n elements.

---

## Sorting with Comparison

```javascript
function topNByRevenue(customers, n) {
  return [...customers]
    .sort((a, b) => b.revenue - a.revenue)
    .slice(0, n)
}
```

**Complexity:** Time O(k log k) where k = customers.length. Space O(k) for the copy.

**What this means:** Sorts all customers even though only n are needed. For k=1M and n=10, this does ~20M comparison operations.

**Can it be improved?** A partial sort or selection algorithm gives O(k log n) or O(k) average. However, sort+slice is clear, correct, and fast enough for most inputs. Only optimise if profiling identifies this as a bottleneck with large k and small n.

---

## Recursive Tree Traversal

```javascript
function countNodes(node) {
  if (!node) return 0
  return 1 + countNodes(node.left) + countNodes(node.right)
}
```

**Complexity:** Time O(n) where n = number of nodes. Space O(h) where h = tree height (call stack depth). Worst case h = n (degenerate tree), best case h = log n (balanced).

**What this means:** Visits every node exactly once. Stack depth is the main concern. For balanced trees this is ~20 frames for 1M nodes. For degenerate trees, may overflow the stack.

**Can it be improved?** An iterative approach with an explicit stack avoids stack overflow risk with the same O(n) time and O(h) space. Slightly less readable but necessary for untrusted input with potentially deep trees.
