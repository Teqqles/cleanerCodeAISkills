# Refactoring: Scala Examples

## Extract Condition Into Named Function

```scala
// Before
if (user.subscriptionEndDate.isAfter(Instant.now())
    && user.plan != "free"
    && !user.suspended) {
  showPremiumContent()
}

// After
def hasActivePaidSubscription(user: User): Boolean =
  user.subscriptionEndDate.isAfter(Instant.now())
    && user.plan != "free"
    && !user.suspended

if (hasActivePaidSubscription(user)) showPremiumContent()
```

## Replace Magic Values With Named Constants

```scala
// Before
if (response.status == 429) {
  Thread.sleep(2000)
  retry(3)
}

// After
val HttpTooManyRequests = 429
val RetryDelayMs        = 2_000L
val MaxRetries          = 3

if (response.status == HttpTooManyRequests) {
  Thread.sleep(RetryDelayMs)
  retry(MaxRetries)
}
```

## Delete Dead Code

```scala
// Before
def calculateShipping(order: Order): BigDecimal = {
  // val legacyRate = BigDecimal("4.99")  // old flat rate
  order.totalWeightKg * BigDecimal("1.5")
}

// After
def calculateShipping(order: Order): BigDecimal =
  order.totalWeightKg * BigDecimal("1.5")
```

## Flatten Nesting: Use Option and Match

```scala
// Before: nested Options and ifs
def getInvoiceTotal(maybeInvoice: Option[Invoice]): BigDecimal = {
  if (maybeInvoice.isDefined) {
    val invoice = maybeInvoice.get
    if (invoice.isPaid) {
      if (invoice.lines.nonEmpty) {
        invoice.lines.map(_.amount).sum
      } else BigDecimal(0)
    } else BigDecimal(0)
  } else BigDecimal(0)
}

// After: flat match
def getInvoiceTotal(maybeInvoice: Option[Invoice]): BigDecimal =
  maybeInvoice match {
    case None                                      => BigDecimal(0)
    case Some(i) if !i.isPaid                      => BigDecimal(0)
    case Some(i) if i.lines.isEmpty                => BigDecimal(0)
    case Some(i)                                   => i.lines.map(_.amount).sum
  }
```
