---
name: algorithmic-complexity
description: Reports Big O time and space complexity for all algorithmically complex code you write or review. Use this skill whenever you write a function that iterates, recurses, sorts, searches, filters, groups, or transforms data structures. Trigger on any code involving loops over collections, recursive calls, nested iterations, hash map construction, tree traversal, graph algorithms, or data pipeline operations. Also trigger when the user asks "how fast is this", "will this scale", "what's the complexity", or "can this be faster". You MUST annotate every non-trivial algorithm with its cost and explain what that cost means in plain terms.
---

# Algorithmic Complexity

You MUST report the Big O time and space complexity of every algorithmically complex piece of code you write or review. This is not optional. The report is part of your response to the user, not a code comment. Present clean code first, then follow it with your complexity analysis as prose. When the cost is non-trivial, you explain what it means and whether it can be improved without sacrificing readability or reusability.

## When to Report Complexity

Report complexity whenever code involves:

- Iteration over a collection (especially nested)
- Sorting, searching, or filtering
- Building data structures (maps, sets, trees, graphs)
- Recursive algorithms
- Data pipeline chains (map/filter/reduce/flatMap)
- Any operation whose cost is not obviously O(1)

Do NOT annotate trivial constant-time operations (field access, single arithmetic, hash lookups in isolation). Do annotate anything where the cost depends on input size.

## What to Report

For every non-trivial algorithm, state:

1. **Time complexity**: how execution time grows with input size
2. **Space complexity**: how memory usage grows with input size
3. **What it means**: a plain-language explanation of the practical cost
4. **Can it be improved?**: whether a better algorithm exists that preserves readability and reusability

## Format

The complexity report is conversational output to the user, NOT a code comment. Never embed complexity annotations inside the code. Instead, present the code cleanly, then report the analysis as prose below the code block in your response to the user.

Structure:

1. Present the code (no complexity comments inside it)
2. After the code block, provide a **Complexity** section in your response:

**Complexity:** Time O(n log n), Space O(n) where n is the number of items.

**What this means:** Sorting dominates at n log n. The lookup map built afterwards is linear and adds linear memory. For 100K items this is roughly 1.7M operations for the sort, fast on any modern machine.

**Can it be improved?** If input arrives pre-sorted, the sort can be skipped for O(n) time. Otherwise this is optimal for comparison-based sorting. No readability sacrifice needed.

Keep the report concise and practical. The user needs to know the cost, understand it intuitively, and know if better is achievable. This is a report to the user, not documentation in the code.

## Big O Reference

State complexity using standard notation. When explaining, translate to what it means for real input sizes:

| Notation | Name | Meaning |
|----------|------|---------|
| O(1) | Constant | Cost does not grow with input size |
| O(log n) | Logarithmic | Halving the problem each step; 1 billion items costs ~30 steps |
| O(n) | Linear | Touch each item once; scales directly with input |
| O(n log n) | Linearithmic | Typical optimal sort; 1M items is ~20M operations |
| O(n^2) | Quadratic | Nested iteration; 10K items means 100M operations. Suspect at scale |
| O(n^3) | Cubic | Triple nesting; rarely acceptable beyond small inputs |
| O(2^n) | Exponential | Doubles with each new element; impractical beyond ~20-30 items |
| O(n!) | Factorial | All permutations; impractical beyond ~10-12 items |

Always contextualise: "O(n^2) where n is the number of orders" is actionable. "O(n^2)" alone is not.

## Amortised and Average Case

When amortised or average-case complexity differs meaningfully from worst case, report both in your response:

**Complexity:** Time O(1) amortised, O(n) worst case (hash table resize). Space O(n).

**What this means:** In practice each insert is constant-time. Occasionally the underlying table resizes, copying all n entries. This happens infrequently enough that the cost per operation averages to O(1).

## Hidden Costs in Pipelines

Chained operations can hide quadratic behaviour:

```
// Each .includes() is O(m), called n times = O(n * m)
const common = listA.filter(item => listB.includes(item))
```

Call this out. Suggest the Set-based alternative:

```
// O(n + m) time, O(m) space
const setB = new Set(listB)
const common = listA.filter(item => setB.has(item))
```

## The Readability Constraint

Never recommend a faster algorithm that makes the code harder to understand or reuse, unless the performance gain is critical and the context demands it. State the tradeoff explicitly:

- "An O(n) solution exists using X, but it's significantly harder to follow. The O(n log n) version is correct here unless profiling shows this is a bottleneck."
- "This can be reduced from O(n^2) to O(n) with a hash set, and the result is equally readable. Prefer the O(n) version."

When the faster version is equally clear: always prefer it.
When the faster version is complex: flag it, explain the tradeoff, default to clarity unless scale demands otherwise.

## Space-Time Tradeoffs

When a faster algorithm requires more memory, state the tradeoff in your report:

**Complexity:** Time O(n), Space O(n) where n is the number of entries.

**What this means:** Builds a hash map of all entries for constant-time lookups. The brute-force alternative is O(n^2) time with O(1) space.

**Can it be improved?** This trades linear memory for linear time. At typical input sizes (<100K items) the memory cost is negligible. The O(n) version is equally readable. Prefer it.

## Recursive Algorithms

For recursive code, state the recurrence and its resolution in your report:

**Complexity:** Time O(2^n) without memoisation, O(n) with memoisation. Space O(n) for the memo table and call stack.

**What this means:** Without memoisation, the same subproblems recompute exponentially. For n=30 that is over a billion calls. With memoisation each subproblem is solved exactly once, reducing to n calls total.

**Can it be improved?** The memoised version is optimal for this recurrence. An iterative bottom-up approach avoids call-stack depth but offers the same O(n) time and space. Both are equally readable; choose based on whether the language handles deep recursion well.

## Language Examples

When the project language is known, read the relevant reference for idiom-specific performance patterns:

- `references/typescript.md`
- `references/python.md`
- `references/java.md`
- `references/scala.md`
- `references/javascript.md`

## Performance Is a Design Concern, Not an Afterthought

Choosing the right algorithm at write time is cheaper than profiling and rewriting later. But premature optimisation that obscures intent is still wrong. The rule: pick the best algorithm that is equally readable. Only reach for a less readable one when the data demands it.
