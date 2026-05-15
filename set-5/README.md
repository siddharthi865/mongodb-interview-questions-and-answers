# Set 5

| S.No. | Question                                                                                            |
| ----- | --------------------------------------------------------------------------------------------------- |
| 1.    | [How does MongoDB handle high availability?](#question-1-how-does-mongodb-handle-high-availability) |

## Question 1. How does MongoDB handle high availability?

MongoDB handles **high availability (HA)** primarily through **Replica Sets**. A replica set is a group of MongoDB servers that maintain the same dataset by replicating data across multiple nodes. This ensures automatic failover, redundancy, and minimal downtime if a server crashes.

## Core Concept: Replica Sets

A typical replica set contains:

- **Primary Node**
  - Accepts all write operations.

- **Secondary Nodes**
  - Replicate data from the primary using the oplog (operation log).
  - Can serve read operations if configured.

- **Arbiter (Optional)**
  - Participates only in elections; stores no data.

### Architecture

```text
        Clients
           |
       Primary
        /    \
 Secondary  Secondary
```

If the primary fails:

1. Secondaries detect the failure.
2. An election occurs automatically.
3. One secondary becomes the new primary.
4. Applications reconnect automatically using the replica set connection string.

### Example: Creating a Replica Set

### Step 1: Start MongoDB Instances

```bash
mongod --replSet rs0 --port 27017 --dbpath /data/db1
mongod --replSet rs0 --port 27018 --dbpath /data/db2
mongod --replSet rs0 --port 27019 --dbpath /data/db3
```

### Explanation

- `--replSet rs0`
  - Defines replica set name.

- Each node runs on a separate port.

### Step 2: Initialize Replica Set

Connect to one node:

```javascript
mongosh --port 27017
```

Initialize:

```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "localhost:27017" },
    { _id: 1, host: "localhost:27018" },
    { _id: 2, host: "localhost:27019" },
  ],
});
```

### Step 3: Check Replica Set Status

```javascript
rs.status();
```

This shows:

- Current primary
- Secondary members
- Replication lag
- Election information

### How Replication Works

MongoDB uses the **oplog** (`local.oplog.rs`) to replicate operations.

### Flow

1. Client writes to primary.
2. Primary records operation in oplog.
3. Secondaries continuously pull oplog entries.
4. Data stays synchronized.

### Automatic Failover

If the primary crashes:

```text
Primary DOWN
   ↓
Election starts
   ↓
Secondary elected as new Primary
```

Failover usually completes in a few seconds.

Applications should use a replica-set-aware connection string:

```javascript
mongodb://host1:27017,host2:27018,host3:27019/?replicaSet=rs0
```

This allows automatic reconnection after failover.

### Read Availability

MongoDB supports **read preferences**:

```javascript
db.employees.find().readPref("secondary");
```

Useful for:

- Analytics queries
- Reporting
- Reducing load on primary

### Performance & Best Practices

### Use Odd Number of Voting Nodes

Example:

- 3 nodes
- 5 nodes

Prevents split-brain issues during elections.

### Avoid Excessive Replication Lag

Monitor:

```javascript
rs.printSecondaryReplicationInfo();
```

### Use Write Concern for Durability

```javascript
db.employees.insertOne({ name: "David" }, { writeConcern: { w: "majority" } });
```

Ensures data is replicated to majority nodes before acknowledgment.

### Common Pitfalls

| Pitfall                     | Issue                              |
| --------------------------- | ---------------------------------- |
| Single-node deployment      | No failover                        |
| Even number of voting nodes | Election problems                  |
| Large replication lag       | Stale reads                        |
| No majority write concern   | Possible data loss during failover |

### Real-World Use Case

An e-commerce application:

- Primary handles orders/payments.
- Secondaries handle analytics/reporting.
- If the primary server fails, another node becomes primary automatically, keeping the application online.

### Follow-up Interview Questions

### 1. What is the difference between replication and sharding?

- **Replication** provides high availability and redundancy.
- **Sharding** provides horizontal scaling by distributing data.

### 2. What is an oplog in MongoDB?

- The oplog is a capped collection storing all write operations used by secondary nodes for replication.
