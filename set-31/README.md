# Set 31

| S.No. | Question                                                                                 |
| ----- | ---------------------------------------------------------------------------------------- |
| 1.    | [Design a real-time dashboard backend](#question-1-design-a-real-time-dashboard-backend) |

## Question 1. Design a real-time dashboard backend

A **real-time dashboard backend** is a system designed to ingest, process, and serve continuously changing data (metrics, logs, events) with **low latency updates** to multiple clients (web/mobile dashboards).

As a senior MongoDB engineer, I’ll design this focusing on **MongoDB-centric architecture**, scalability, and real-time capabilities.

### 1. Core Idea (Interview Definition)

A real-time dashboard backend is a system that:

- Collects streaming data (events, metrics, logs)
- Processes and aggregates it in near real-time
- Serves updated results to UI dashboards with minimal delay (sub-second to few seconds)

Typical use cases:

- E-commerce live orders dashboard
- System monitoring (CPU, memory, logs)
- Financial trading dashboards
- IoT telemetry dashboards

### 2. High-Level Architecture

#### Components

1. **Data Producers**
   - Apps, services, IoT devices
   - Emit events (orders, clicks, metrics)

2. **Ingestion Layer**
   - Kafka / RabbitMQ / HTTP ingestion API
   - Buffers high throughput streams

3. **Processing Layer**
   - Stream processors (Kafka Streams / Node.js / Spark Streaming)
   - Aggregates metrics in real-time

4. **MongoDB (Core Storage + Aggregation Engine)**
   - Stores raw + aggregated data
   - Uses:
     - Time-series collections
     - Aggregation pipelines
     - Change Streams

5. **Real-time Push Layer**
   - WebSockets / SSE (Server-Sent Events)
   - Push updates to frontend dashboards

### 3. MongoDB Data Model (Optimized for Dashboard)

#### 3.1 Raw Events Collection (Time-Series)

```javascript
db.events.insertOne({
  type: "order_created",
  value: 1200,
  region: "IN",
  createdAt: new Date(),
});
```

#### Optimized as Time-Series Collection (recommended)

```javascript
db.createCollection("events", {
  timeseries: {
    timeField: "createdAt",
    metaField: "type",
    granularity: "seconds",
  },
});
```

Why:

- Efficient compression
- Fast range queries
- Optimized for dashboards

#### 3.2 Aggregated Metrics Collection

```javascript
{
  metric: "orders_per_minute",
  value: 450,
  windowStart: ISODate("2026-05-14T10:00:00Z"),
  windowEnd: ISODate("2026-05-14T10:01:00Z")
}
```

### 4. Real-Time Aggregation Pipeline

#### Example: Orders per minute

```javascript
db.events.aggregate([
  {
    $match: {
      type: "order_created",
    },
  },
  {
    $group: {
      _id: {
        $dateTrunc: {
          date: "$createdAt",
          unit: "minute",
        },
      },
      count: { $sum: 1 },
      revenue: { $sum: "$value" },
    },
  },
  {
    $sort: { _id: -1 },
  },
]);
```

#### Explanation

- `$match` → filters relevant event type
- `$dateTrunc` → groups into time buckets (minute-level aggregation)
- `$sum` → computes KPIs
- `$sort` → ensures latest data first

### 5. Real-Time Updates using Change Streams

MongoDB Change Streams enable real-time push without polling.

```javascript
const pipeline = [{ $match: { "fullDocument.type": "order_created" } }];

const changeStream = db.events.watch(pipeline);

changeStream.on("change", (next) => {
  console.log("New event:", next.fullDocument);
  // push to WebSocket clients
});
```

#### Why this is powerful

- No polling
- Sub-second latency
- Native MongoDB feature (replica set required)

### 6. Backend API Layer (Node.js Example)

#### WebSocket server

```javascript
io.on("connection", (socket) => {
  console.log("Client connected");

  changeStream.on("change", (data) => {
    socket.emit("dashboard_update", {
      metric: "orders",
      value: data.fullDocument.value,
    });
  });
});
```

### 7. Indexing Strategy (VERY IMPORTANT)

#### Critical indexes

```javascript
db.events.createIndex({ type: 1, createdAt: -1 });
```

#### Why

- Fast filtering by event type
- Efficient time-range queries
- Prevents collection scans

### For aggregation-heavy workloads

```javascript
db.events.createIndex({ createdAt: -1 });
```

### 8. Performance Optimizations

#### 8.1 Use Time-Series Collections

- 3–10x better performance for time-based data

#### 8.2 Pre-aggregation (Materialized Views Pattern)

Instead of computing every request:

```javascript
db.dashboard_metrics.updateOne(
  { metric: "orders_per_minute" },
  { $inc: { value: 1 } },
  { upsert: true },
);
```

#### 8.3 Windowed Aggregation Strategy

- 1s / 10s / 1 min windows depending on dashboard granularity

#### 8.4 Sharding (for scale)

Shard key recommendation:

```javascript
{ createdAt: 1, type: 1 }
```

Ensures:

- Time-based distribution
- Even load across shards

### 9. Real-Time Flow Summary

1. Event generated (e.g., order placed)
2. Ingested via API/Kafka
3. Stored in MongoDB time-series collection
4. Change Stream triggers update
5. Aggregation pipeline computes metrics
6. WebSocket pushes update to dashboard UI

### 10. Common Pitfalls

### ❌ Polling database every second

→ causes heavy load

### ❌ No indexing on time fields

→ slow dashboards

### ❌ Over-aggregation in real-time

→ increases CPU pressure

### ❌ Using regular collections for time-series data

→ poor performance at scale

### 11. Follow-up Interview Questions

#### Q1: How would you scale this system to millions of events per second?

Answer:

- Kafka ingestion layer
- MongoDB sharding on time + region
- Pre-aggregation using stream processors
- Horizontal WebSocket scaling with Redis pub/sub

#### Q2: How do Change Streams differ from polling?

Answer:

- Change Streams are oplog-based (efficient)
- No repeated queries needed
- Lower latency and DB load
- Requires replica set / sharded cluster
