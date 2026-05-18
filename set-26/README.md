# Set 26

| S.No. | Question                                                                                               |
| ----- | ------------------------------------------------------------------------------------------------------ |
| 1.    | [Design a container metadata tracking system](#question-1-design-a-container-metadata-tracking-system) |

## Question 1. Design a container metadata tracking system

### 1. Core Idea (Interview Definition)

A **container metadata tracking system** stores, manages, and queries lifecycle and operational metadata of containers (like Docker/Kubernetes containers). It tracks attributes such as container ID, image version, node placement, status, resource usage, and events (start/stop/restart).

MongoDB is ideal here because container metadata is:

- Highly **write-heavy & time-series-like**
- Semi-structured (fields vary by container type)
- Requires fast querying by **containerId, cluster, status, timestamps**

### 2. Schema Design (MongoDB Collection)

We design a single collection: `containers`

```javascript
use interview_db;

db.containers.insertOne({
  containerId: "c123",
  name: "auth-service",
  image: "auth:v2.1",
  cluster: "prod-cluster-1",
  node: "node-7",
  status: "RUNNING",
  createdAt: ISODate("2026-05-14T10:00:00Z"),
  updatedAt: ISODate("2026-05-14T10:10:00Z"),

  resources: {
    cpu: 0.5,
    memoryMB: 512
  },

  network: {
    ip: "10.2.3.4",
    ports: [8080, 9090]
  },

  events: [
    { type: "START", timestamp: ISODate("2026-05-14T10:00:00Z") },
    { type: "HEALTH_CHECK", timestamp: ISODate("2026-05-14T10:05:00Z") }
  ]
});
```

### 3. Design Explanation (Step-by-Step)

#### Embedded vs Referenced Design

- We **embed events inside container docs** because:
  - Events are tightly coupled
  - Usually queried with container

- If events grow unbounded → move to `container_events` collection (scalability tradeoff)

### 4. Indexing Strategy (Critical for Performance)

#### Primary Index (default)

```javascript
// _id index (automatic)
```

#### Recommended Indexes

```javascript
db.containers.createIndex({ containerId: 1 }, { unique: true });

db.containers.createIndex({ cluster: 1, status: 1 });

db.containers.createIndex({ node: 1 });

db.containers.createIndex({ updatedAt: -1 });
```

#### Why?

- `containerId`: fast lookup (main identifier)
- `cluster + status`: monitoring dashboards (e.g., RUNNING containers per cluster)
- `node`: debugging node-level failures
- `updatedAt`: recent activity tracking

### 5. Common Queries

#### Find running containers in a cluster

```javascript
db.containers.find({
  cluster: "prod-cluster-1",
  status: "RUNNING",
});
```

#### Get container history (events)

```javascript
db.containers.findOne({ containerId: "c123" }, { events: 1, _id: 0 });
```

### 6. Scaling Considerations

#### Sharding Strategy

Shard by:

```javascript
{ cluster: 1, containerId: 1 }
```

Why?

- Ensures even distribution across clusters
- Prevents hotspotting on single cluster

#### Time-series optimization (optional)

If events grow huge:

- Move to `container_events` collection
- Use **time-based sharding**

### 7. Performance Pitfalls

- ❌ Unbounded `events` array → document growth limit (16MB)
- ❌ Missing compound indexes → slow dashboard queries
- ❌ Frequent updates on large documents → contention

### 8. Advanced Enhancement (Real-world)

#### Change Streams for real-time tracking

```javascript
db.containers.watch([
  { $match: { operationType: { $in: ["insert", "update"] } } },
]);
```

Used for:

- Live dashboards
- Alerting system
- Auto-healing infrastructure

### 9. Follow-up Interview Questions

#### Q1: How would you handle millions of container events per second?

Move events to a separate sharded time-series collection.

#### Q2: How do you ensure consistency when container status changes rapidly?

Use atomic `$set` updates with optimistic concurrency (version field).
