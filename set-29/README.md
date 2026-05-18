# Set 29

| S.No. | Question                                                                                                         |
| ----- | ---------------------------------------------------------------------------------------------------------------- |
| 1.    | [Design a real-time transaction monitoring system](#question-1-design-a-real-time-transaction-monitoring-system) |

## Question 1. Design a real-time transaction monitoring system

A **real-time transaction monitoring system** is designed to ingest, analyze, and flag financial transactions as they happen (or near real-time) to detect fraud, AML (anti–money laundering) violations, anomalies, or risk patterns.

### 1. Core Requirement Understanding

#### Functional Requirements

- Ingest transactions in real time (payments, transfers, withdrawals)
- Detect fraud/anomalies (rule-based + ML scoring)
- Alert system for suspicious transactions
- Maintain transaction history + audit trail
- Support user/account risk scoring
- Query recent transactions quickly

#### Non-Functional Requirements

- Low latency (≤ 200–500 ms per transaction evaluation)
- High throughput (10K–1M TPS scale)
- High availability (multi-region)
- Strong auditability (immutable logs)
- Horizontal scalability

### 2. High-Level Architecture

#### Flow

```text
Payment Gateway
      ↓
Kafka / Event Stream
      ↓
Stream Processing Layer (MongoDB Change Streams / Kafka Streams / Flink)
      ↓
Risk Engine (Rules + ML)
      ↓
MongoDB (Transaction Store + Risk Store)
      ↓
Alerting System (Kafka / Webhooks / Email / Dashboard)
```

### 3. MongoDB Data Model Design

#### 3.1 Transactions Collection

```javascript
{
  _id: ObjectId(),
  txnId: "TXN12345",
  userId: ObjectId("..."),
  amount: 1200,
  currency: "INR",
  type: "UPI_TRANSFER",
  status: "PENDING", // APPROVED | DECLINED | FLAGGED
  timestamp: ISODate(),
  merchantId: ObjectId(),
  location: {
    city: "Gurgaon",
    country: "IN"
  },
  deviceId: "device_xyz",
  riskScore: 82
}
```

#### Indexing Strategy

```javascript
db.transactions.createIndex({ userId: 1, timestamp: -1 });
db.transactions.createIndex({ txnId: 1 }, { unique: true });
db.transactions.createIndex({ riskScore: -1 });
```

**Why:**

- userId + timestamp → fast user history lookup
- riskScore → fraud dashboards
- txnId → idempotency

#### 3.2 User Risk Profile Collection

```javascript
{
  userId: ObjectId(),
  avgTxnAmount: 1500,
  velocityPerHour: 12,
  riskLevel: "MEDIUM",
  lastUpdated: ISODate()
}
```

#### 3.3 Fraud Rules Collection (Dynamic Rules Engine)

```javascript
{
  ruleId: "RULE_HIGH_VALUE",
  condition: {
    amount: { $gt: 10000 }
  },
  riskWeight: 50,
  action: "FLAG"
}
```

### 4. Real-Time Processing Design

#### Step 1: Ingestion Layer

- All transactions pushed to **Kafka topic: `transactions`**
- Ensures durability + replay capability

#### Step 2: Stream Processing (Core Logic)

##### Option A: MongoDB Change Streams

Used if MongoDB is primary write store

```javascript
const changeStream = db.transactions.watch();

changeStream.on("change", (change) => {
  const txn = change.fullDocument;

  if (txn.amount > 10000) {
    db.transactions.updateOne(
      { _id: txn._id },
      { $set: { status: "FLAGGED", riskScore: 90 } },
    );
  }
});
```

##### Option B (Preferred at scale): Kafka + Flink

- Stateless + stateful computations
- Window-based fraud detection

Example logic:

- 5 transactions in 1 minute → velocity fraud
- geo mismatch detection
- device fingerprint anomaly

#### Step 3: Risk Scoring Engine

##### Rule-based scoring

```javascript
function calculateRisk(txn, userProfile) {
  let score = 0;

  if (txn.amount > 10000) score += 40;
  if (userProfile.velocityPerHour > 10) score += 30;
  if (txn.deviceId.isNew) score += 20;

  return score;
}
```

#### Step 4: MongoDB Update Pattern

```javascript
db.transactions.updateOne(
  { txnId: "TXN12345" },
  {
    $set: {
      riskScore: 75,
      status: "FLAGGED",
    },
  },
);
```

### 5. Scalability Design (Important Interview Section)

#### 5.1 Sharding Strategy

Shard key:

```javascript
{
  userId: "hashed";
}
```

Why:

- Even distribution
- Avoid hotspotting (high-transaction users)

#### 5.2 Read Optimization

- Secondary indexes for analytics
- Read from secondary nodes for dashboards

#### 5.3 Write Optimization

- Bulk inserts from Kafka consumers
- Avoid heavy aggregations in write path

### 6. Fraud Detection Enhancements

#### 6.1 Aggregation Pipeline (Velocity Detection)

```javascript
db.transactions.aggregate([
  {
    $match: {
      userId: ObjectId("..."),
      timestamp: { $gte: ISODate("2026-05-14T00:00:00Z") },
    },
  },
  {
    $group: {
      _id: "$userId",
      txnCount: { $sum: 1 },
      totalAmount: { $sum: "$amount" },
    },
  },
]);
```

#### 6.2 Time-window fraud detection (real-world use case)

- 1 min spike detection
- Geo-location mismatch
- Device switching

### 7. Alerting System

- Kafka topic: `fraud_alerts`
- Consumers:
  - Email service
  - SMS gateway
  - Admin dashboard

Example alert doc:

```javascript
{
  txnId: "TXN12345",
  reason: "High velocity detected",
  severity: "HIGH"
}
```

### 8. Audit & Compliance

- Transactions are **immutable (append-only pattern)**
- Use soft updates for status only
- Maintain full history via:
  - Change Streams OR
  - Event sourcing collection

### 9. Performance Optimizations

### Key techniques

- Index compound fields (userId + timestamp)
- TTL indexes for temporary fraud logs
- Pre-aggregation for dashboards
- Avoid `$lookup` in real-time pipeline

### 10. Common Pitfalls

- Overloading MongoDB with heavy stream processing
- Missing proper shard key → hotspotting
- Not handling idempotency in transaction ingestion
- Lack of replay mechanism (Kafka solves this)

### 11. Follow-up Interview Questions

#### Q1: How would you ensure exactly-once processing?

Use Kafka idempotent producers + MongoDB unique txnId index + upsert operations.

#### Q2: How would you scale fraud detection to millions of TPS?

Move compute to distributed stream processing (Flink/Kafka Streams), keep MongoDB as state store only, shard by hashed userId, and pre-aggregate risk signals.
