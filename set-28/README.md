# Set 28

| S.No. | Question                                                                                             |
| ----- | ---------------------------------------------------------------------------------------------------- |
| 1.    | [Design tenant-level rate limiting tracking](#question-1-design-tenant-level-rate-limiting-tracking) |

## Question 1. Design tenant-level rate limiting tracking

### 1. Core Concept (Interview Definition)

Tenant-level rate limiting ensures that each **tenant (customer/org/user group)** is restricted to a defined number of requests per time window (e.g., 1000 requests/minute), preventing abuse and ensuring fair usage.

In MongoDB, we typically implement this using a **high-write, low-latency counter tracking system**, often with atomic updates and time-windowed documents.

### 2. Requirements Breakdown

#### Functional Requirements

- Track API requests per tenant
- Enforce limits per time window (per second/minute/hour)
- Support multiple plans (free, pro, enterprise)
- Allow near real-time enforcement

#### Non-functional Requirements

- Extremely low latency (sub-millisecond reads/writes)
- High write throughput
- Horizontally scalable
- Minimal contention under heavy load

### 3. Data Model (MongoDB Schema)

#### Option A: Time-window bucketed counter (Recommended)

```javascript
db.rate_limits.insertOne({
  tenantId: "tenant_123",
  windowStart: ISODate("2026-05-14T10:00:00Z"),
  windowType: "MINUTE",
  count: 42,
  limit: 1000,
  updatedAt: ISODate(),
});
```

#### Key Idea

- One document per tenant per time window
- `windowStart` aligns to fixed intervals (e.g., minute buckets)

### 4. Core Rate Limiting Logic

#### Atomic Increment with Upsert

```javascript
db.rate_limits.updateOne(
  {
    tenantId: "tenant_123",
    windowStart: ISODate("2026-05-14T10:00:00Z"),
  },
  {
    $inc: { count: 1 },
    $setOnInsert: {
      limit: 1000,
      windowType: "MINUTE",
      createdAt: new Date(),
    },
    $set: {
      updatedAt: new Date(),
    },
  },
  { upsert: true },
);
```

#### How it works

- `$inc` → atomically increments request count
- `$setOnInsert` → initializes metadata only once
- `upsert: true` → creates document if not exists

### 5. Enforcement Logic

After update, fetch or use return value:

```javascript
const doc = db.rate_limits.findOne({
  tenantId: "tenant_123",
  windowStart: ISODate("2026-05-14T10:00:00Z"),
});

if (doc.count > doc.limit) {
  throw new Error("Rate limit exceeded");
}
```

In production, this is often combined into a **single atomic operation using findOneAndUpdate**

### 6. Optimized Atomic Enforcement (Best Practice)

```javascript
db.rate_limits.findOneAndUpdate(
  {
    tenantId: "tenant_123",
    windowStart: ISODate("2026-05-14T10:00:00Z"),
    count: { $lt: 1000 },
  },
  {
    $inc: { count: 1 },
    $setOnInsert: {
      limit: 1000,
      windowType: "MINUTE",
    },
    $set: { updatedAt: new Date() },
  },
  {
    upsert: true,
    returnDocument: "after",
  },
);
```

#### Why this is important

- Prevents race conditions
- Ensures atomic check + increment
- Avoids double counting under concurrency

### 7. Indexing Strategy

```javascript
db.rate_limits.createIndex({ tenantId: 1, windowStart: 1 }, { unique: true });
```

#### Why

- Ensures fast lookup per tenant per window
- Enforces uniqueness to prevent duplicates
- Critical for high-throughput APIs

### 8. Window Calculation Strategy

Example (minute-based window):

```javascript
function getWindowStart(date) {
  date.setSeconds(0, 0);
  return date;
}
```

Alternative:

- Fixed window (simple, slightly inaccurate at edges)
- Sliding window (more accurate, more complex)
- Token bucket (best UX, usually implemented in Redis)

### 9. Scaling Considerations

#### Problem

- Hot tenants → high contention on same document

#### Solutions

##### 1. Shard by tenantId

```javascript
sh.shardCollection("rate_limits", { tenantId: "hashed" });
```

##### 2. Sub-window splitting (for heavy tenants)

```javascript
tenantId + windowStart + shardKey;
```

##### 3. Hybrid approach (MongoDB + Redis)

- Redis for real-time counters
- MongoDB for persistence/audit

### 10. Cleanup Strategy

Old windows must be removed:

```javascript
db.rate_limits.createIndex({ windowStart: 1 }, { expireAfterSeconds: 3600 });
```

### 11. Real-World Use Cases

- API gateways (Stripe-style rate limits)
- SaaS tenant quotas
- Fraud prevention systems
- Multi-tenant microservices

### 12. Common Pitfalls

- ❌ Not using atomic updates → race conditions
- ❌ Large document growth per tenant → performance degradation
- ❌ No indexing on (tenantId, windowStart)
- ❌ Using naive client-side counting (unsafe)
- ❌ Ignoring clock drift in distributed systems

### 13. Follow-up Interview Questions

#### Q1: How would you implement sliding window rate limiting in MongoDB?

Use multiple window buckets + aggregation over last N intervals.

#### Q2: Why might Redis be preferred over MongoDB for rate limiting?

Redis provides in-memory atomic counters with lower latency, but MongoDB provides durability and auditability.
