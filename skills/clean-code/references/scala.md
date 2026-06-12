# Clean Code: Scala Examples

## Naming

```scala
// Bad
val d = LocalDate.now()
val arr = users.filter(_.a > 18)

// Good
val currentDate = LocalDate.now()
val adultUsers = users.filter(_.age > 18)
```

## Flat Logic: Prefer Option and Match Over Null Guards

Scala uses `Option` instead of null checks. Pattern matching replaces nested if/else.

```scala
// Bad: nested null checks and if/else
def getDiscount(user: User): BigDecimal = {
  if (user != null) {
    if (user.isPremium) {
      if (user.yearsActive > 2) BigDecimal("0.2") else BigDecimal("0.1")
    } else BigDecimal("0")
  } else BigDecimal("0")
}

// Good: flat match on Option
def getDiscount(maybeUser: Option[User]): BigDecimal =
  maybeUser match {
    case None                                 => BigDecimal(0)
    case Some(user) if !user.isPremium        => BigDecimal(0)
    case Some(user) if user.yearsActive > 2   => BigDecimal("0.2")
    case _                                    => BigDecimal("0.1")
  }
```

## Single-Purpose Functions

```scala
// Bad: validates, saves, and sends email in one function
def processUser(user: User): Unit = {
  if (user.email.isEmpty) throw new IllegalArgumentException("missing email")
  userRepository.save(user)
  emailService.sendWelcome(user.email)
}

// Good: each function does one thing; errors expressed as Either
def validateUser(user: User): Either[String, User] =
  if (user.email.isEmpty) Left("email is required") else Right(user)

def registerUser(user: User, db: UserRepository, email: EmailService): Either[String, Unit] =
  validateUser(user).map { valid =>
    db.save(valid)
    email.sendWelcome(valid.email)
  }
```

## Domain-Centric Structure

```
src/main/scala/
├── order/
│   ├── Order.scala
│   ├── PlaceOrderUseCase.scala
│   └── OrderSpec.scala
├── payment/
│   ├── Payment.scala
│   └── ProcessPaymentUseCase.scala
└── user/
    ├── User.scala
    └── RegisterUserUseCase.scala
```
