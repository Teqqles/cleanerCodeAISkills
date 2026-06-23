# Algorithmic Complexity: Scala Examples

## Hidden Quadratic with List.contains

```scala
def removeDuplicates(items: List[String]): List[String] =
  items.foldLeft(List.empty[String]) { (acc, item) =>
    if (acc.contains(item)) acc else acc :+ item
  }
```

**Complexity:** Time O(n^2) where n = items.length. Space O(n) for the accumulator.

**What this means:** `acc.contains` is O(n) on a List, called n times. Additionally `:+` on List is O(n). Combined: O(n^2).

**Can it be improved?** Yes, equally readable:

```scala
def removeDuplicates(items: List[String]): List[String] =
  items.distinct
```

**Complexity:** Time O(n), Space O(n). Uses a HashSet internally while preserving order. Strictly better.

---

## Nested Lookup Replaced by Set

```scala
def findMatching(orders: List[Order], customerIds: List[String]): List[Order] =
  orders.filter(order => customerIds.contains(order.customerId))
```

**Complexity:** Time O(n * m) where n = orders, m = customerIds. Space O(n) for output.

**Can it be improved?**

```scala
def findMatching(orders: List[Order], customerIds: List[String]): List[Order] = {
  val idSet = customerIds.toSet
  orders.filter(order => idSet.contains(order.customerId))
}
```

**Complexity:** Time O(n + m), Space O(m) for the Set. Equally readable.

---

## GroupBy and MapValues

```scala
def ordersByCustomer(orders: List[Order]): Map[String, List[Order]] =
  orders.groupBy(_.customerId)
```

**Complexity:** Time O(n), Space O(n) where n = orders.length.

**What this means:** Single pass over all orders, inserting each into a hash-map bucket. Each insertion is O(1) amortised.

**Can it be improved?** No. Every element must be visited at least once.

---

## Tail-Recursive Accumulation

```scala
def sumLargerThan(items: List[Int], threshold: Int): Long = {
  @annotation.tailrec
  def loop(remaining: List[Int], acc: Long): Long = remaining match {
    case Nil => acc
    case head :: tail => loop(tail, if (head > threshold) acc + head else acc)
  }
  loop(items, 0L)
}
```

**Complexity:** Time O(n), Space O(1) stack frames (tail-recursive eliminates stack growth). O(n) for the input list itself.

**What this means:** Processes each element exactly once with constant overhead per step. Optimal.

**Can it be improved?** No. This is a single linear pass. The tail recursion compiles to a loop.

---

## Collection Choice Matters

```scala
// Vector: O(log32 n) ~ effectively O(1) indexed access
val byIndex = vector(50000)

// List: O(n) indexed access
val byIndex = list(50000)
```

**What this means:** Scala's immutable List is a linked list. Indexed access is linear. Vector provides effectively constant-time random access via a trie. Choose Vector when indexed access is needed; List when you only prepend and traverse.
