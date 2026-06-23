---
name: loose-coupling
description: Enforces loose coupling as a non-negotiable design constraint. Use this skill whenever you are writing or reviewing code that involves multiple modules, classes, or services communicating with each other. Trigger when the user says "how do I make this extensible", "this is hard to change", "how do I add a new type without touching everything", or when you notice concrete dependencies, long call chains, modules that break when unrelated code changes, or systems that cannot be tested without standing up the world. Tight coupling is the single largest barrier to reuse, maintainability, and extension.
---

# Loose Coupling

You treat loose coupling as a non-negotiable constraint. Tight coupling is the single largest barrier to reuse, maintainability, and extension. Code exists to be reused, maintained, and extended without imposing burden on the rest of the system.

## The Cost of Coupling

Every dependency between two modules is a constraint on both. Tight coupling means:

- Changing one module forces changes in others
- Testing one module requires standing up its dependencies
- Reusing one module means dragging its dependants along
- Extending the system means modifying existing, working code

Loose coupling means each module can be understood, tested, replaced, and extended independently.

## Depend on Abstractions, Never on Concretions

- Business logic depends on interfaces, not concrete implementations
- The interface is owned by the consumer, defined by what it needs, not what the provider happens to offer
- A module that imports a concrete class from another module is coupled to that module's implementation decisions
- When in doubt, ask: "can I replace this dependency with a fake in a test without changing this code?"

## Law of Demeter: Talk Only to Your Immediate Collaborators

Do not reach through objects to talk to their internals.

- A method should only call methods on: its own object, its parameters, objects it creates, its direct dependencies
- `order.getCustomer().getAddress().getCity()` couples you to the entire chain
- Fix: ask the object to do the work: `order.shippingCity()` or pass what you need directly
- Long chains are a symptom of responsibilities in the wrong place

**Example:**
```
// Coupled: knows Order's internal structure
function getShippingLabel(order) {
  return order.getCustomer().getAddress().format()
}

// Decoupled: Order owns the decision
function getShippingLabel(order) {
  return order.shippingLabel()
}
```

## Tell, Don't Ask

Do not interrogate an object for its state and then make decisions outside it. Tell the object what to do.

- "Ask" couples the caller to the object's internal representation
- "Tell" lets the object change its internals without affecting callers
- If you find yourself getting data from an object just to pass it somewhere else, the behaviour belongs on that object

**Example:**
```
// Ask: coupled to Timer's internals
if (timer.getElapsed() > timer.getTimeout()) {
  timer.stop()
  alert()
}

// Tell: Timer owns timeout logic
timer.onTimeout(() => alert())
```

## Interface Segregation: Narrow Interfaces Over Broad Ones

- No client should depend on methods it does not use
- Split broad interfaces into focused ones: `Readable`, `Writable` not `ReadWriteStore`
- A class that implements a broad interface couples all its consumers to capabilities they do not need
- Narrow interfaces are easier to implement, easier to fake, easier to replace

## Events and Messages Over Direct Calls

When two components need to communicate but should not know about each other:

- Use events, callbacks, or message passing instead of direct method calls
- The producer emits; consumers subscribe. Neither knows the other exists
- Adding a new consumer requires zero changes to the producer
- This is the Observer pattern applied architecturally

## No Shared Mutable State Between Modules

Shared mutable state is implicit coupling that does not appear in any interface.

- Modules that read and write the same global state are coupled through that state
- Changes to the state's structure break every module that touches it
- Fix: each module owns its own state and exposes it through a defined interface
- If modules must share data, share immutable values or communicate through messages

## Extension Without Modification

The system should be extendable by adding new code, not by editing existing code.

- New behaviour arrives as new implementations of existing interfaces
- Plugin architectures, strategy patterns, and event systems enable this
- If adding a new variant requires editing a switch statement, the design is coupled to the set of variants
- Open/Closed Principle: open for extension, closed for modification

**Example:**
```
// Coupled: every new payment method edits this function
function process(payment) {
  if (payment.type === 'card') { ... }
  else if (payment.type === 'bank') { ... }
  else if (payment.type === 'crypto') { ... }
}

// Decoupled: new payment methods are new classes
interface PaymentProcessor {
  process(payment): Receipt
}

// Adding 'crypto' means adding CryptoProcessor, not editing existing code
```

## Measure Coupling by Asking Three Questions

For every dependency between two modules:

1. **Can I test this module without the other?** If no, coupling is too tight.
2. **Can I replace the other module without changing this one?** If no, coupling is too tight.
3. **Can I reuse this module in a different context?** If no, coupling is too tight.

If any answer is no, introduce an abstraction at the boundary.

## Language Examples

When the project language is known, read the relevant reference for decoupling patterns and idioms:

- `references/typescript.md`
- `references/python.md`
- `references/java.md`
- `references/scala.md`
- `references/javascript.md`

## Coupling Is the Enemy of Change

Every design decision should reduce the cost of future change. Loose coupling is not a preference or a style: it is the structural prerequisite for code that can be reused, maintained, and extended without imposing burden on the rest of the system.
