# Set 27

| S.No. | Question                                                                                     |
| ----- | -------------------------------------------------------------------------------------------- |
| 1.    | [Design anti-cheat event logging system](#question-1-design-anti-cheat-event-logging-system) |

## Question 1. Design anti-cheat event logging system

### Core Idea (Interview Definition)

An **anti-cheat event logging system** is a real-time, high-throughput system that collects, stores, and analyzes gameplay/user activity signals (clicks, movement, trades, login patterns, device info) to detect suspicious or malicious behavior such as bots, exploits, or fraud.

It must support:

- Massive write throughput (millions of events/sec)
- Low-latency ingestion
- Flexible schema (new cheat signals evolve frequently)
- Fast querying for investigations + real-time detection pipelines

### 1. High-Level Architecture

#### Components

1. **Client/Game Servers**
   - Emit events: `move`, `shoot`, `trade`, `login`, `click`, etc.

2. **Ingestion Layer (Kafka / Event Queue)**
   - Buffers spikes
   - Ensures durability

3. **Event Processor (Stream layer)**
   - Enrichment (IP → geo, device fingerprint)
   - Rule evaluation (heuristics / ML features)

4. **MongoDB Cluster (Core storage)**
   - Stores raw + processed events

5. **Detection Engine**
   - Aggregates patterns (velocity hacks, bot behavior)
   - Writes “flags”

6. **Admin Dashboard / Investigation API**

### 2. MongoDB Schema Design

We design for **append-heavy workload + flexible schema**.

#### Collection: `player_events`

```javascript
{
  _id: ObjectId(),
  playerId: ObjectId("..."),
  sessionId: "sess_12345",

  eventType: "movement", // login, attack, trade, movement
  timestamp: ISODate("2026-05-14T10:10:00Z"),

  // flexible payload for different event types
  payload: {
    x: 120,
    y: 450,
    speed: 85,
    action: "jump"
  },

  deviceInfo: {
    ip: "192.168.1.10",
    deviceId: "device_hash_abc",
    os: "Android"
  },

  flags: {
    suspicious: false,
    reasons: []
  }
}
```

#### Why this schema works

- `payload` allows **schema evolution**
- `eventType` enables filtering
- `deviceInfo` supports fraud detection
- `flags` supports inline annotation

### 3. Indexing Strategy (VERY IMPORTANT)

#### Primary indexes

```javascript
db.player_events.createIndex({ playerId: 1, timestamp: -1 });
```

#### Why

- Fast per-player timeline lookup (investigations)

```javascript
db.player_events.createIndex({ eventType: 1, timestamp: -1 });
```

#### Why

- Analyze patterns per event type (e.g., all trades)

```javascript
db.player_events.createIndex({ "deviceInfo.deviceId": 1 });
```

#### Why

- Detect multi-account cheating

#### TTL Index (optional for cost control):

```javascript
db.player_events.createIndex(
  { timestamp: 1 },
  { expireAfterSeconds: 60 * 60 * 24 * 30 }, // 30 days
);
```

### 4. Write Flow (High Throughput)

#### Step-by-step

1. Client emits event
2. Kafka buffers it
3. Stream processor enriches it
4. Batch insert into MongoDB

#### Bulk insert example

```javascript
db.player_events.insertMany([
  {
    playerId: ObjectId("64a..."),
    eventType: "movement",
    timestamp: new Date(),
    payload: { x: 10, y: 20, speed: 120 },
    deviceInfo: { ip: "1.1.1.1", deviceId: "abc" },
  },
  {
    playerId: ObjectId("64a..."),
    eventType: "movement",
    timestamp: new Date(),
    payload: { x: 15, y: 25, speed: 130 },
  },
]);
```

#### Why insertMany

- Reduces network overhead
- Improves write throughput significantly

### 5. Detection Queries (Real-time & Batch)

#### Example: Speed Hack Detection

```javascript
db.player_events.aggregate([
  { $match: { eventType: "movement" } },

  {
    $group: {
      _id: "$playerId",
      maxSpeed: { $max: "$payload.speed" },
      avgSpeed: { $avg: "$payload.speed" },
    },
  },

  {
    $match: {
      maxSpeed: { $gt: 100 },
    },
  },
]);
```

#### Explanation

- `$match`: filters movement events
- `$group`: computes per-player stats
- `$match`: flags abnormal speed

#### Example: Multi-account detection (same device)

```javascript
db.player_events.aggregate([
  {
    $group: {
      _id: "$deviceInfo.deviceId",
      uniquePlayers: { $addToSet: "$playerId" },
    },
  },
  {
    $project: {
      count: { $size: "$uniquePlayers" },
    },
  },
  {
    $match: {
      count: { $gt: 3 },
    },
  },
]);
```

#### Insight

- Same device → multiple accounts → bot farm

### 6. Performance Optimization

#### Use

- **Sharding by `playerId`**

```javascript
sh.shardCollection("game.player_events", { playerId: "hashed" });
```

#### Why hashed shard key?

- Even distribution
- Avoid hot partitions

#### Use capped pre-aggregation

- Move heavy analytics to **separate collection**

#### Avoid

- Large unbounded arrays inside documents
- Frequent updates on same document (write contention)

### 7. Real-time Alerting Pipeline

1. Stream processor detects anomaly
2. Writes to `cheat_flags`

   ```javascript
   {
   playerId: ObjectId(),
   type: "speed_hack",
   severity: "HIGH",
   timestamp: ISODate()
   }
   ```

3. Admin dashboard consumes flags

### 8. Key Trade-offs

| Choice                  | Trade-off                                          |
| ----------------------- | -------------------------------------------------- |
| MongoDB flexible schema | Easy evolution vs larger storage footprint         |
| TTL indexing            | Cost savings vs loss of historical data            |
| Sharding by playerId    | Balanced load vs cross-player analytics complexity |
| Denormalized events     | Fast reads vs duplication                          |

### 9. Production Enhancements

- Change Streams → real-time detection
- Atlas Search → pattern-based fraud queries
- ML pipeline → feature extraction from event streams
- Compression (WiredTiger) for storage efficiency

### 10. Follow-up Interview Questions

#### 1. How would you handle schema evolution in event-heavy systems?

Use `payload` flexible schema + version field (`schemaVersion`) + backward-compatible parsers.

#### 2. How do you prevent MongoDB from becoming a write bottleneck?

Use:

- Sharding
- Bulk inserts
- Kafka buffering
- Avoid hot shard keys
- Pre-aggregation pipelines
