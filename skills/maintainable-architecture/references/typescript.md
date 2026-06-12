# Maintainable Architecture: TypeScript Examples

## Layer Separation

```
src/
├── domain/               # no framework imports: pure TypeScript
│   ├── order/
│   │   ├── Order.ts
│   │   ├── OrderRepository.ts     # interface
│   │   └── PlaceOrderUseCase.ts
│   └── payment/
│       ├── Payment.ts
│       └── PaymentGateway.ts      # interface
├── infrastructure/        # implements domain interfaces
│   ├── postgres/
│   │   └── PostgresOrderRepository.ts
│   └── stripe/
│       └── StripePaymentGateway.ts
└── api/                   # framework (Express, NestJS, etc.)
    └── OrderController.ts
```

## Domain Types Have No Framework Dependencies

```typescript
// domain/order/Order.ts: no imports from Express, TypeORM, or any framework
export class Order {
  constructor(
    readonly id: string,
    readonly customerId: string,
    readonly items: ReadonlyArray<LineItem>
  ) {}

  total(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0)
  }
}
```

## Map at the Boundary

```typescript
// infrastructure/postgres/PostgresOrderRepository.ts
export class PostgresOrderRepository implements OrderRepository {
  async findById(id: string): Promise<Order | null> {
    const row = await this.db.query('SELECT * FROM orders WHERE id = $1', [id])
    if (!row) return null
    return this.toDomain(row)          // map ORM row to domain type here
  }

  private toDomain(row: OrderRow): Order {
    return new Order(row.id, row.customer_id, row.items.map(toLineItem))
  }
}
```

## Composition Over Inheritance

```typescript
// Bad: extending for discount behaviour
class PremiumOrder extends Order {
  total(): number { return super.total() * 0.8 }
}

// Good: injected discount policy
class Order {
  constructor(
    readonly items: ReadonlyArray<LineItem>,
    private readonly discountPolicy: DiscountPolicy
  ) {}

  total(): number {
    return this.discountPolicy.apply(
      this.items.reduce((sum, item) => sum + item.price, 0)
    )
  }
}
```
