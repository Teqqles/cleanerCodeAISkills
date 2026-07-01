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


## Don't abuse matching by hiding cyclomatic complexity and values in play.
Sometimes matching can become very unwieldy, due to the growth of potential combinations.
Logical branching should be made clear. Matching can hide cyclomatic complexity, so use judiciously.

```scala
sealed trait UpdateType

case object StatusUpdate extends UpdateType

case object ContactDetailsUpdate extends UpdateType

sealed trait Status
case object Pending extends Status
case object Failed extends Status
case object Succeeded extends Status

case class Update(
  updateType: UpdateType,
  maybeNewStatus: Option[Status],
  maybeFailedStatusComment: Option[String],
  maybeSucceedStatusComment: Option[String],
  maybeNewFirstName: Option[String],
  maybeNewLastName: Option[String]
)

val update = Update(
  updateType = StatusUpdate,
  maybeNewStatus = None,
  maybeFailedStatusComment = None,
  maybeSucceedStatusComment = None,
  maybeNewFirstName = None,
  maybeNewLastName = None
)


// Business requirements are hidden within the flat structure. A good way to hide bugs. Bugs hide in incoherence.
//
// Branching is done on StatusUpdate, but made obscure by flattening logic like this. Needless puzzle
// Makes knowing what these values represent harder to keep track of. Requires needless memory work.
// People tend to follow the path of least resistance, so every field/business rule gets added can make
// it descend into combinatorial explosion in the default clause. 100s of combinations that need to be calculated to
// debug what it may fail on. Breaks Linus's law due to the high mount of proactive effort to understand, so people
// will only try to understand it if they are made to.
update match {
  case Update(StatusUpdate, Some(Pending), None, None, None, None) =>
    true

  case Update(StatusUpdate, Some(Failed), Some(_), None, None, None) =>
    true

  case Update(StatusUpdate, Some(Succeeded), None, Some(_), None, None) =>
    true


  //Clumsy way to do an or on firstname and lastname
  case Update(ContactDetailsUpdate, None, None, None, Some(_), _) =>
    true

  case Update(ContactDetailsUpdate, None, None, None, _, Some(_)) =>
    true

  case _ =>
    false
}

// Easily skimmable
update.updateType match {
  case StatusUpdate =>
    if (update.maybeNewFirstName.isDefined || update.maybeNewLastName.isDefined) {
      false
    } else {
      update.maybeNewStatus match {
        case Some(newStatus) =>
          newStatus match {
            case Pending =>
              update.maybeFailedStatusComment.isEmpty && update.maybeSucceedStatusComment.isEmpty

            case Failed =>
              update.maybeFailedStatusComment.isDefined && update.maybeSucceedStatusComment.isEmpty

            case Succeeded =>
              update.maybeFailedStatusComment.isEmpty && update.maybeSucceedStatusComment.isDefined
          }
        case None =>
          false
      }
    }

  case ContactDetailsUpdate =>
    val doesNotHaveUnexpectedValues = update.maybeNewStatus.isEmpty &&
      update.maybeFailedStatusComment.isEmpty &&
      update.maybeSucceedStatusComment.isEmpty

    doesNotHaveUnexpectedValues && (update.maybeNewFirstName.isDefined || update.maybeNewLastName.isDefined)

}



```