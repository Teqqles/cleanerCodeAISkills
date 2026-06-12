---
name: refactoring
description: Applies safe, continuous refactoring practices to improve code structure without changing behaviour. Use this skill whenever the user asks you to refactor, clean up, simplify, or improve existing code. Trigger when the user says "this is getting messy", "clean this up", "simplify this", "rename this", or "remove dead code". Also trigger when you spot magic values, overly long functions, deep nesting, duplicated logic, or misleading names in code you are reviewing: suggest the refactoring even if not asked.
---

# Refactoring

You refactor continuously and safely. These practices apply to all languages and codebases.

## Refactoring Is a Habit, Not a Task

Every time you touch code, leave it clearer than you found it: the Boy Scout Rule.

- When adding a feature, refactor the surrounding code to make the addition clean
- When fixing a bug, improve the code that made the bug possible
- "We'll clean this up later" is a deferral to never

## Small, Reversible Steps Only

Large refactors are hard to review, easy to break, and hard to revert.

- Each step leaves the tests green
- Commit after each meaningful step: a commit is a checkpoint
- One step per commit when a refactor requires multiple steps
- When something goes wrong, revert to the last green commit

## Always Rely on a Passing Test Suite

- Never refactor without tests covering the code being changed
- Run the tests after every step: not at the end
- Write tests before refactoring if none exist
- The test suite proves behaviour has not changed

## Remove Dead Code Immediately

Dead code confuses every reader who must determine whether it matters.

- Delete unused variables, parameters, imports, functions, and classes
- Do not comment out code: version control preserves history
- Do not write `// TODO: remove this`: remove it now or track it in an issue
- Unused code compounds; readers assume it might be important

## Simplify Complex Logic Into Named Functions

- Extract complex conditionals into a function named after the decision: `isEligibleForDiscount()` not `if (age > 65 && tier == 2)`
- Extract logic into named functions when the name adds meaning, even if used once
- Each function operates at one level of abstraction

## Replace Magic Values With Named Constants

No numeric or string literals with non-obvious meaning in logic.

- `MAX_RETRY_ATTEMPTS = 3` is clear; `if retries > 3` is not
- Name the constant after its meaning: `MAX_CONNECTIONS`, not `THREE`
- Group related constants together

**Example:**
```
// Before
if (response.status == 429) {
  sleep(2000)
  retry(3)
}

// After
const HTTP_TOO_MANY_REQUESTS = 429
const RETRY_DELAY_MS = 2000
const MAX_RETRIES = 3

if (response.status == HTTP_TOO_MANY_REQUESTS) {
  sleep(RETRY_DELAY_MS)
  retry(MAX_RETRIES)
}
```

## Language Examples

When the project language is known, read the relevant reference for extract-condition, magic-value, dead-code, and nesting examples:

- `references/typescript.md`
- `references/python.md`
- `references/java.md`
- `references/scala.md`: nesting examples use `Option` + pattern match
- `references/javascript.md`

## Refactoring Must Not Alter Observable Behaviour

A refactor changes structure, not behaviour.

- Behaviour changes get their own commit, separate from refactoring
- Mixing the two makes review impossible and corrupts the safety net
- "Refactor then change" is always cleaner than "change and refactor simultaneously"
