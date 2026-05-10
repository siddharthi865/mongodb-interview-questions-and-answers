# Set 14

| S.No. | Question                                                                                                                    |
| ----- | --------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you read from secondary nodes in a replica set?](#question-1-how-do-you-read-from-secondary-nodes-in-a-replica-set) |

## Question 1. How do you read from secondary nodes in a replica set?

**Core concept:**
In a MongoDB replica set, secondary nodes replicate data asynchronously from the primary via the oplog. By default, all **writes go to the primary**, but **reads can be routed to secondaries** using _read preference settings_.

This is mainly used to:

- Scale read traffic horizontally
- Reduce load on the primary
- Serve analytics/reporting workloads with slightly stale data tolerance

## How to Read from Secondary Nodes

MongoDB provides **Read Preference** settings to control where reads are routed.

### 1. Using Mongo Shell (mongosh)

```javascript
// Set read preference to secondary
db.getMongo().setReadPref("secondary");

// Now all queries prefer secondary nodes
db.employees.find({ department: "Engineering" });
```

### Line-by-line

- `getMongo()` → gets underlying connection
- `setReadPref("secondary")` → instructs driver to route reads to secondaries
- `find()` → query now executes on secondary nodes

### 2. Query-Level Read Preference

```javascript
db.employees.find(
  { department: "Engineering" },
  { readPreference: "secondary" },
);
```

- Applies only to this query
- Overrides global/default settings

### 3. Connection String (Driver Level)

```bash
mongodb://localhost:27017,localhost:27018,localhost:27019/interview_db?replicaSet=rs0&readPreference=secondary
```

- `replicaSet=rs0` → enables replica set awareness
- `readPreference=secondary` → routes reads to secondaries

### 4. Driver Example (Node.js style)

```javascript
const client = new MongoClient(uri, {
  readPreference: "secondary",
});
```

## How It Works Internally

1. Primary receives all writes
2. Writes are recorded in the **oplog**
3. Secondaries replicate oplog entries asynchronously
4. When read preference = `secondary`, driver:
   - Selects an eligible secondary
   - Routes query to it
   - Returns potentially slightly stale data

## Important Pitfalls

### 1. Stale Reads (Eventual Consistency)

Secondaries may lag behind primary → outdated data.

### 2. No Strong Consistency Guarantee

If you need strict consistency, use:

```javascript
readPreference: "primary";
```

### 3. Causal Consistency Caveat

Even with sessions, reading from secondaries can break causal consistency unless carefully configured.

### 4. Failover Behavior

If no secondaries are available:

- Query may fail (depending on settings like `secondaryPreferred`)

## Best Practice

- Use `secondary` for:
  - Analytics dashboards
  - Reporting systems
  - Background processing

- Avoid for:
  - Financial transactions
  - Real-time user-facing reads

## Related Settings

- **readPreference modes:**
  - `primary`
  - `secondary`
  - `primaryPreferred`
  - `secondaryPreferred`
  - `nearest`

- **readConcern:**
  - `local` (default)
  - `majority` (safer consistency)
  - `linearizable` (strongest, primary only)

## Follow-up Interview Questions

**1. What is the difference between `secondary` and `secondaryPreferred`?**
`secondary` strictly uses secondaries only, while `secondaryPreferred` falls back to primary if secondaries are unavailable.

**2. How does replication lag affect read consistency?**
Reads from secondaries may return stale data due to oplog replication delay, leading to eventual consistency behavior.
