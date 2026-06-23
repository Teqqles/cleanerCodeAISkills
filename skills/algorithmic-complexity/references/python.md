# Algorithmic Complexity: Python Examples

## Hidden Quadratic in List Membership

```python
def remove_duplicates(items: list[str]) -> list[str]:
    result = []
    for item in items:
        if item not in result:
            result.append(item)
    return result
```

**Complexity:** Time O(n^2) where n = len(items). Space O(n) for result.

**What this means:** `item not in result` scans the list linearly each time. For 10K items, up to 50M comparisons.

**Can it be improved?** Yes, equally readable:

```python
def remove_duplicates(items: list[str]) -> list[str]:
    return list(dict.fromkeys(items))
```

**Complexity:** Time O(n), Space O(n). Dict preserves insertion order and provides O(1) membership. Strictly better.

---

## Nested Loop Replaced by Set

```python
def find_common(list_a: list[str], list_b: list[str]) -> list[str]:
    return [x for x in list_a if x in list_b]
```

**Complexity:** Time O(n * m) where n = len(list_a), m = len(list_b). Space O(min(n, m)) for output.

**Can it be improved?**

```python
def find_common(list_a: list[str], list_b: list[str]) -> list[str]:
    set_b = set(list_b)
    return [x for x in list_a if x in set_b]
```

**Complexity:** Time O(n + m), Space O(m) for the set. Equally readable, eliminates the hidden quadratic.

---

## Sorting vs Heap for Top-N

```python
def top_n_by_revenue(customers: list[Customer], n: int) -> list[Customer]:
    return sorted(customers, key=lambda c: c.revenue, reverse=True)[:n]
```

**Complexity:** Time O(k log k) where k = len(customers). Space O(k) for the sorted copy.

**Can it be improved?**

```python
import heapq

def top_n_by_revenue(customers: list[Customer], n: int) -> list[Customer]:
    return heapq.nlargest(n, customers, key=lambda c: c.revenue)
```

**Complexity:** Time O(k log n), Space O(n). `heapq.nlargest` maintains a heap of size n.

**What this means:** For k=1M and n=10, this is ~20M vs ~7M operations. Both are readable; prefer `nlargest` when n is much smaller than k.

---

## Dictionary Construction

```python
def index_by_email(users: list[User]) -> dict[str, User]:
    return {user.email: user for user in users}
```

**Complexity:** Time O(n), Space O(n) where n = len(users).

**What this means:** Each dict insertion is O(1) amortised. This is optimal. Every element must be visited.

**Can it be improved?** No. Linear time is the lower bound for building an index.

---

## Recursive Fibonacci

```python
def fib(n: int) -> int:
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
```

**Complexity:** Time O(2^n), Space O(n) for call stack depth.

**What this means:** For n=40, roughly 1 billion calls. Impractical beyond n~35.

**Can it be improved?**

```python
from functools import cache

@cache
def fib(n: int) -> int:
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
```

**Complexity:** Time O(n), Space O(n). Each subproblem computed once. Equally readable with a single decorator.
