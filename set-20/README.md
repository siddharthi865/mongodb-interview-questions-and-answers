# Set 20

| S.No. | Question                                                                                                                                                |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you implement multi-region replication for low-latency reads?](#question-1-how-do-you-implement-multi-region-replication-for-low-latency-reads) |

## Question 1. How do you implement multi-region replication for low-latency reads?

Multi-region replication in MongoDB is typically implemented using a **Replica Set distributed across multiple geographic regions**. The goal is:

- **Low-latency reads** by serving users from the nearest region
- **High availability** during regional outages
- **Disaster recovery**
- **Data redundancy**

MongoDB replica sets support this natively by replicating data asynchronously from the **Primary** to multiple **Secondary** nodes across regions.

### Example Architecture

| Region    | Nodes               | Purpose                  |
| --------- | ------------------- | ------------------------ |
| Mumbai    | Primary + Secondary | Main write region        |
| Singapore | Secondary           | APAC low-latency reads   |
| Frankfurt | Secondary           | Europe low-latency reads |

Clients connect using **Read Preferences** to read from nearby secondaries.

### Replica Set Configuration

Example replica set:

```javascript
rs.initiate({
  _id: "global-rs",
  members: [
    {
      _id: 0,
      host: "mumbai-primary:27017",
      priority: 2,
    },
    {
      _id: 1,
      host: "singapore-secondary:27017",
      priority: 1,
    },
    {
      _id: 2,
      host: "frankfurt-secondary:27017",
      priority: 1,
    },
  ],
});
```

### Explanation

#### `rs.initiate()`

Initializes the replica set.

#### `_id: "global-rs"`

Defines the replica set name.

#### Members

Each node is added as a member.

```javascript
{
  _id: 1,
  host: "singapore-secondary:27017",
  priority: 1
}
```

- `host` → region-specific server
- `priority` → election priority

Higher priority nodes are preferred as primary.

### Read Preference for Low-Latency Reads

Applications should use:

```javascript
db.getMongo().setReadPref("nearest");
```

Or in connection string:

```javascript
mongodb://host1,host2,host3/?readPreference=nearest
```

This allows reads from the geographically closest healthy node.

### Real-World Example

Suppose:

- Users in Europe read from Frankfurt secondary
- Users in Asia read from Singapore secondary
- Writes still go to Mumbai primary

Result:

- Lower read latency
- Reduced cross-region traffic
- Faster user experience

### Tag-Aware Reads (Recommended)

MongoDB supports **tag sets** for region-aware routing.

#### Configure Tags

```javascript
cfg = rs.conf();

cfg.members[0].tags = { region: "india" };
cfg.members[1].tags = { region: "apac" };
cfg.members[2].tags = { region: "eu" };

rs.reconfig(cfg);
```

#### Read from Specific Region

```javascript
db.getMongo().setReadPref("secondaryPreferred", [{ region: "eu" }]);
```

This ensures European clients read only from EU nodes.

### Performance Considerations

#### 1. Replication Lag

Cross-region replication is asynchronous.

Possible issue:

- Secondary may lag behind primary

Check lag:

```javascript
rs.printSecondaryReplicationInfo();
```

#### 2. Read Concern

For globally distributed systems:

```javascript
db.collection.find().readConcern("majority");
```

Ensures data is majority committed.

#### 3. Write Concern

Use:

```javascript
db.collection.insertOne({ name: "Alice" }, { writeConcern: { w: "majority" } });
```

This improves durability across regions.

### Best Practices

#### Use Hidden Analytics Nodes

For reporting:

```javascript
{
  host: "analytics-node:27017",
  hidden: true,
  priority: 0
}
```

Prevents elections and isolates analytics traffic.

#### Use Delayed Nodes for DR

```javascript
secondaryDelaySecs: 3600;
```

Provides rollback protection.

#### Use MongoDB Atlas Global Clusters

MongoDB Atlas provides managed:

- Geo-distributed clusters
- Zone sharding
- Automatic regional failover
- Localized reads

This is the preferred enterprise solution.

### Common Pitfalls

| Pitfall                      | Issue                         |
| ---------------------------- | ----------------------------- |
| Reading stale data           | Secondary lag                 |
| Too many cross-region writes | High latency                  |
| Incorrect priorities         | Unwanted elections            |
| Nearest reads without tags   | Reads may route unpredictably |

### Follow-Up Interview Questions

#### 1. What is the difference between `nearest` and `secondaryPreferred`?

- `nearest` → reads from lowest-latency node
- `secondaryPreferred` → prefers secondaries but can read from primary

#### 2. How does MongoDB Atlas improve multi-region deployments?

Atlas provides:

- Global clusters
- Automated failover
- Regional data placement
- Managed backups
- Geo-sharding support

These simplify globally distributed architectures significantly.
