# Set 32

| S.No. | Question                                                                          |
| ----- | --------------------------------------------------------------------------------- |
| 1.    | [Design a coupon & discount engine](#question-1-design-a-coupon--discount-engine) |

## Question 1. Design a coupon & discount engine

A **coupon & discount engine** is a system that validates, applies, and manages promotional rules (like coupons, percentage discounts, fixed discounts, BOGO, etc.) during checkout while ensuring correctness, performance, and scalability.

It must answer:

- Is this coupon valid?
- Can this user use it?
- What discount applies to this cart?
- How do we prevent misuse (double redemption, expiry, limits)?

## 1. Schema Design (MongoDB)

We design collections to keep rules flexible and query-efficient.

### coupons collection

```js
{
  _id: ObjectId(),
  code: "SAVE20",
  type: "PERCENTAGE", // FIXED, PERCENTAGE, BOGO
  value: 20,          // 20%
  minCartValue: 500,
  maxDiscount: 200,

  validFrom: ISODate("2026-01-01"),
  validTo: ISODate("2026-12-31"),

  usageLimit: 1000,        // global limit
  usageCount: 120,

  perUserLimit: 1,

  applicableCategories: ["electronics", "fashion"],
  excludedProducts: [],

  isActive: true
}
```

#### coupon_redemptions collection (critical for tracking)

```js
{
  _id: ObjectId(),
  couponCode: "SAVE20",
  userId: ObjectId("..."),
  orderId: ObjectId("..."),
  redeemedAt: ISODate()
}
```

This prevents duplicate usage and enables auditability.

#### orders collection (simplified)

```js
{
  _id: ObjectId(),
  userId: ObjectId(),
  items: [
    { productId: 1, category: "electronics", price: 300 }
  ],
  totalAmount: 300,
  couponCode: "SAVE20",
  discountApplied: 60,
  finalAmount: 240
}
```

### 2. Coupon Validation Flow

#### Step-by-step

1. Check coupon exists & active
2. Validate date range
3. Check cart minimum value
4. Check user usage limits
5. Check global usage limit
6. Ensure product/category eligibility
7. Apply discount logic
8. Record redemption atomically

### 3. MongoDB Query Examples

#### Find valid coupon

```js
db.coupons.findOne({
  code: "SAVE20",
  isActive: true,
  validFrom: { $lte: new Date() },
  validTo: { $gte: new Date() },
});
```

#### 🔎 Explanation

- `$lte / $gte` ensures time validity
- `isActive` prevents soft-deleted coupons

#### Check user redemption count

```js
db.coupon_redemptions.countDocuments({
  couponCode: "SAVE20",
  userId: ObjectId("USER_ID"),
});
```

#### Apply category filtering in cart (aggregation example)

```js
db.coupons.aggregate([
  { $match: { code: "SAVE20", isActive: true } },
  {
    $project: {
      eligible: {
        $in: ["electronics", "$applicableCategories"],
      },
      value: 1,
      type: 1,
    },
  },
]);
```

### 4. Discount Calculation Logic

#### Types

#### 1. Percentage Discount

```bash
discount = min(cartTotal * value/100, maxDiscount)
```

#### 2. Fixed Discount

```bash
discount = value
```

#### 3. BOGO (Buy One Get One)

Handled at item-level via matching logic in cart pipeline.

### 5. Atomicity & Concurrency (VERY IMPORTANT)

To avoid double redemption:

```js
db.coupons.updateOne(
  {
    code: "SAVE20",
    usageCount: { $lt: "$usageLimit" },
  },
  {
    $inc: { usageCount: 1 },
  },
);
```

Then insert redemption record.

In production:

- Use **transactions (replica set required)**

```js
session.startTransaction();
```

### 6. Indexing Strategy

#### Critical indexes

```js
db.coupons.createIndex({ code: 1 }, { unique: true });

db.coupons.createIndex({ isActive: 1, validFrom: 1, validTo: 1 });

db.coupon_redemptions.createIndex({ couponCode: 1, userId: 1 });

db.coupon_redemptions.createIndex({ userId: 1 });
```

#### Why?

- Fast lookup by code
- Efficient validation window filtering
- Prevent duplicate user redemption checks

### 7. Scalability Considerations

#### Problem: High traffic coupon rush (flash sale)

Solutions:

- Cache coupon metadata in Redis
- Pre-validate coupons at API gateway
- Use **atomic counters** for usage limits
- Shard redemptions by `couponCode` or `userId`

### 8. Fraud Prevention

- Limit per IP/user/device
- Track redemption patterns
- Block suspicious bursts
- Ensure idempotent checkout using `orderId`

### 9. Real-world Enhancements

- Tiered coupons (VIP users)
- Stackable discounts rules engine
- Geo-based coupons
- Time-window flash coupons
- A/B testing discount effectiveness

### Common Pitfalls

- ❌ Not handling concurrency → coupon overuse
- ❌ Missing indexes on `code`
- ❌ Calculating discounts client-side only
- ❌ Not tracking redemption history

### Follow-up Interview Questions

#### 1. How would you prevent race conditions during coupon redemption?

Use MongoDB transactions + atomic `$inc` updates with conditional filters.

#### 2. How would you scale this for millions of daily checkouts?

Cache coupon metadata in Redis, shard redemption logs, and pre-validate at edge/API gateway.
