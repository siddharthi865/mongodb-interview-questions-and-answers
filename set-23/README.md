# Set 23

| S.No. | Question                                                                                                |
| ----- | ------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you implement counters for analytics?](#question-1-how-do-you-implement-counters-for-analytics) |

## Question 1. How do you implement counters for analytics?

Implementing counters for analytics in MongoDB typically means tracking metrics such as page views, likes, clicks, downloads, API hits, or event counts efficiently and atomically.

In interviews, emphasize these key points:

- Use `$inc` for atomic increments.
- Design for high write throughput.
- Consider aggregation or bucket patterns for time-series analytics.
- Handle contention in high-traffic systems.

### Basic Counter Implementation

Suppose we want to track profile views per employee.

```javascript
use interview_db;

// analytics collection
db.analytics.insertOne({
  employeeId: ObjectId("665111111111111111111111"),
  profileViews: 0,
  downloads: 0,
  lastUpdated: new Date()
});
```

Increment the counter:

```javascript
db.analytics.updateOne(
  { employeeId: ObjectId("665111111111111111111111") },
  {
    $inc: { profileViews: 1 },
    $set: { lastUpdated: new Date() },
  },
  { upsert: true },
);
```

### Line-by-Line Explanation

#### `updateOne()`

Updates a single analytics document.

#### Filter

```javascript
{ employeeId: ObjectId(...) }
```

Finds the analytics record for that employee.

#### `$inc`

```javascript
$inc: {
  profileViews: 1;
}
```

Atomically increments the counter.

MongoDB guarantees document-level atomicity, so concurrent updates are safe.

Example:

| Before | After |
| ------ | ----- |
| 100    | 101   |

No race condition occurs.

#### `$set`

Updates metadata like timestamp.

#### `upsert: true`

Creates the document automatically if it doesn’t exist.

### Why `$inc` Is Important

`$inc` is highly optimized and avoids:

❌ Read → Modify → Write race conditions

Bad approach:

```javascript
let doc = db.analytics.findOne(...);
doc.profileViews += 1;
db.analytics.save(doc);
```

This can lose updates under concurrency.

Correct approach:

```javascript
$inc;
```

### Time-Based Analytics Counters

For analytics dashboards, counters are often bucketed by time.

Example: daily page views.

```javascript
db.dailyAnalytics.updateOne(
  {
    employeeId: ObjectId("665111111111111111111111"),
    date: ISODate("2026-05-15"),
  },
  {
    $inc: {
      profileViews: 1,
      downloads: 1,
    },
  },
  { upsert: true },
);
```

Example document:

```javascript
{
  employeeId: ObjectId("665111111111111111111111"),
  date: ISODate("2026-05-15"),
  profileViews: 5400,
  downloads: 320
}
```

This pattern is common in:

- Ad-tech
- Gaming telemetry
- API monitoring
- Social media analytics

### High-Scale Counter Design (Hot Document Problem)

A single counter document can become a bottleneck under massive traffic.

Example:

- Millions of likes on one post
- Heavy write contention

Solution: **Sharded/Distributed Counters**

Example:

```javascript
{
  postId: 101,
  shard: 1,
  likes: 500
}
```

```javascript
{
  postId: 101,
  shard: 2,
  likes: 450
}
```

Increment randomly:

```javascript
db.postCounters.updateOne(
  {
    postId: 101,
    shard: Math.floor(Math.random() * 10),
  },
  {
    $inc: { likes: 1 },
  },
  { upsert: true },
);
```

Read total:

```javascript
db.postCounters.aggregate([
  {
    $match: { postId: 101 },
  },
  {
    $group: {
      _id: "$postId",
      totalLikes: { $sum: "$likes" },
    },
  },
]);
```

This reduces contention dramatically.

### Performance Optimizations

#### 1. Index Frequently Queried Fields

```javascript
db.analytics.createIndex({ employeeId: 1 });
```

For time-series:

```javascript
db.dailyAnalytics.createIndex({
  employeeId: 1,
  date: 1,
});
```

#### 2. Use Retryable Writes

MongoDB supports retryable writes in modern drivers, helping prevent lost increments during transient failures.

#### 3. Avoid Large Growing Documents

Do not store huge arrays of events in one document.

Bad:

```javascript
views: [ ... millions ... ]
```

Instead:

- Store raw events separately
- Maintain aggregated counters

### Real-World Use Cases

- YouTube view counters
- Instagram likes
- API rate tracking
- Gaming scoreboards
- Ecommerce product impressions
- IoT telemetry aggregation

### Common Pitfalls

#### 1. Hotspotting

Too many writes to one document.

Solution:

- Distributed counters
- Sharding

#### 2. Counter Drift

Analytics counters may differ from raw events.

Solution:

- Periodic reconciliation jobs
- Event sourcing architecture

### Follow-Up Interview Questions

#### 1. Why are counters atomic in MongoDB?

MongoDB provides single-document atomicity, and `$inc` modifies the value directly inside the storage engine.

#### 2. How would you implement real-time analytics at massive scale?

Use:

- Kafka/event streams
- MongoDB distributed counters
- Time-series collections
- Aggregation pipelines
- Sharding on entity ID or time bucket
