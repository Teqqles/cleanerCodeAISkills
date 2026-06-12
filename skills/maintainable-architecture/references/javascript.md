# Maintainable Architecture: JavaScript Examples

## Layer Separation

```
src/
├── domain/               # no framework imports: pure JS
│   ├── order/
│   │   ├── Order.js
│   │   ├── OrderRepository.js     # JSDoc interface contract
│   │   └── placeOrder.js
│   └── payment/
│       ├── Payment.js
│       └── PaymentGateway.js      # JSDoc interface contract
├── infrastructure/        # implements domain contracts
│   ├── postgres/
│   │   └── PostgresOrderRepository.js
│   └── stripe/
│       └── StripePaymentGateway.js
└── api/                   # framework (Express, Fastify, etc.)
    └── orderRouter.js
```

## Domain Types Have No Framework Dependencies

```javascript
// domain/order/Order.js: no Express, Sequelize, or framework imports
class Order {
  constructor(id, customerId, items) {
    this.id = id
    this.customerId = customerId
    this.items = items
  }

  total() {
    return this.items.reduce((sum, item) => sum + item.price, 0)
  }
}
```

## Map at the Boundary

```javascript
// infrastructure/postgres/PostgresOrderRepository.js
class PostgresOrderRepository {
  async findById(id) {
    const row = await this.db.query('SELECT * FROM orders WHERE id = $1', [id])
    if (!row) return null
    return this.toDomain(row)       // map DB row to domain type here
  }

  toDomain(row) {
    return new Order(row.id, row.customer_id, row.items.map(toLineItem))
  }
}
```

## Composition Over Inheritance

```javascript
// Bad: extending for discount behaviour
class PremiumOrder extends Order {
  total() { return super.total() * 0.8 }
}

// Good: injected discount policy
class Order {
  constructor(items, discountPolicy) {
    this.items = items
    this.discountPolicy = discountPolicy
  }

  total() {
    const subtotal = this.items.reduce((sum, item) => sum + item.price, 0)
    return this.discountPolicy.apply(subtotal)
  }
}
```
