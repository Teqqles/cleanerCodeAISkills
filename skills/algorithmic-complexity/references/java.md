# Algorithmic Complexity: Java Examples

## Hidden Quadratic with List.contains()

```java
public List<String> removeDuplicates(List<String> items) {
    List<String> result = new ArrayList<>();
    for (String item : items) {
        if (!result.contains(item)) {
            result.add(item);
        }
    }
    return result;
}
```

**Complexity:** Time O(n^2) where n = items.size(). Space O(n) for result.

**What this means:** `result.contains()` is a linear scan, called n times. For 10K items, up to 50M comparisons.

**Can it be improved?** Yes, equally readable:

```java
public List<String> removeDuplicates(List<String> items) {
    return new ArrayList<>(new LinkedHashSet<>(items));
}
```

**Complexity:** Time O(n), Space O(n). LinkedHashSet provides O(1) insertion with insertion-order iteration. Strictly better.

---

## Stream Pipeline with Nested Search

```java
public List<Order> findMatchingOrders(List<Order> orders, List<String> customerIds) {
    return orders.stream()
        .filter(order -> customerIds.contains(order.getCustomerId()))
        .toList();
}
```

**Complexity:** Time O(n * m) where n = orders, m = customerIds. Space O(n) for output.

**Can it be improved?**

```java
public List<Order> findMatchingOrders(List<Order> orders, List<String> customerIds) {
    Set<String> idSet = new HashSet<>(customerIds);
    return orders.stream()
        .filter(order -> idSet.contains(order.getCustomerId()))
        .toList();
}
```

**Complexity:** Time O(n + m), Space O(m) for the Set. Equally readable, eliminates the hidden quadratic.

---

## Sorting for Top-N

```java
public List<Customer> topNByRevenue(List<Customer> customers, int n) {
    return customers.stream()
        .sorted(Comparator.comparingDouble(Customer::getRevenue).reversed())
        .limit(n)
        .toList();
}
```

**Complexity:** Time O(k log k) where k = customers.size(). Space O(k) for the sorted intermediate.

**What this means:** Sorts all customers even though only n are needed. Stream's sorted() must materialise the full stream before limit() applies.

**Can it be improved?** A PriorityQueue of size n gives O(k log n) time:

```java
public List<Customer> topNByRevenue(List<Customer> customers, int n) {
    PriorityQueue<Customer> heap = new PriorityQueue<>(
        Comparator.comparingDouble(Customer::getRevenue));
    for (Customer c : customers) {
        heap.offer(c);
        if (heap.size() > n) heap.poll();
    }
    return new ArrayList<>(heap);
}
```

**Complexity:** Time O(k log n), Space O(n). More code but significantly faster when k >> n. Use when profiling shows the sort is a bottleneck.

---

## HashMap Construction

```java
public Map<String, User> indexByEmail(List<User> users) {
    Map<String, User> map = new HashMap<>(users.size());
    for (User user : users) {
        map.put(user.getEmail(), user);
    }
    return map;
}
```

**Complexity:** Time O(n) amortised, Space O(n) where n = users.size().

**What this means:** Each put is O(1) amortised. Pre-sizing the HashMap avoids resizes. This is optimal.

**Can it be improved?** No. Linear time is the lower bound for indexing n elements.
