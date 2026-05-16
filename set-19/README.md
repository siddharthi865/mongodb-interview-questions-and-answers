# Set 19

| S.No. | Question                                                                                                                                  |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you handle large-scale migrations between collections?](#question-1-how-do-you-handle-large-scale-migrations-between-collections) |

## Question 1. How do you handle large-scale migrations between collections?

Handling large-scale migrations between collections in MongoDB requires careful planning to avoid downtime, data inconsistency, and performance bottlenecks. In interviews, focus on **migration strategy, batching, consistency, rollback, and performance optimization**.

### Core Approach

A production-grade migration usually involves:

1. **Schema/Data Transformation**
2. **Batch Processing**
3. **Incremental Migration**
4. **Validation**
5. **Rollback Strategy**
6. **Zero-Downtime Cutover**

Common scenarios:

- Moving data to a new schema version
- Splitting a large collection
- Merging collections
- Archiving historical data
- Re-indexing with a redesigned structure

### Example: Migrating Employees to a New Collection Structure

#### Source Collection

```javascript
use interview_db;

db.employees.insertMany([
  {
    name: "Alice",
    department: "Engineering",
    salary: 75000,
    skills: ["JavaScript", "React"]
  },
  {
    name: "Bob",
    department: "Engineering",
    salary: 80000,
    skills: ["Node.js", "MongoDB"]
  }
]);
```

### Step 1: Create Target Collection

Suppose we want a new schema:

```javascript
{
  (fullName, dept, compensation, skillCount);
}
```

### Step 2: Use Aggregation + $merge

```javascript
db.employees.aggregate([
  {
    $project: {
      fullName: "$name",
      dept: "$department",
      compensation: "$salary",
      skillCount: { $size: "$skills" },
    },
  },
  {
    $merge: {
      into: "employees_v2",
      whenMatched: "replace",
      whenNotMatched: "insert",
    },
  },
]);
```

### Line-by-Line Explanation

#### `$project`

Transforms document structure.

```javascript
fullName: "$name";
```

Maps `name` → `fullName`.

```javascript
skillCount: {
  $size: "$skills";
}
```

Calculates derived field during migration.

#### `$merge`

Writes transformed documents into target collection.

```javascript
into: "employees_v2";
```

Destination collection.

```javascript
whenMatched: "replace";
```

Replaces existing documents if `_id` matches.

```javascript
whenNotMatched: "insert";
```

Inserts new records.

### Why $merge Is Preferred

`$merge` is safer and scalable because it:

- Supports incremental migrations
- Works efficiently in sharded clusters
- Allows resumable migrations
- Avoids exporting/importing files

MongoDB recommends `$merge` for aggregation-based migrations in modern versions.

### Handling Very Large Collections

For multi-million/billion document collections:

#### 1. Process in Batches

```javascript
let batchSize = 1000;

while (true) {
  const batch = db.employees.find().limit(batchSize).toArray();

  if (batch.length === 0) break;

  const transformed = batch.map((doc) => ({
    insertOne: {
      document: {
        fullName: doc.name,
        dept: doc.department,
      },
    },
  }));

  db.employees_v2.bulkWrite(transformed);

  batchSize += 1000;
}
```

#### Why batching matters

- Prevents memory exhaustion
- Reduces replication lag
- Avoids long-running locks
- Improves checkpointing/recovery

### Production Migration Strategy

#### Blue-Green Migration Pattern

A common enterprise approach:

1. Create new collection (`employees_v2`)
2. Dual-write application changes
3. Backfill old data
4. Validate counts/checksums
5. Switch reads to new collection
6. Retire old collection

This minimizes downtime.

### Performance Optimizations

#### Create Indexes AFTER Migration

Instead of maintaining indexes during writes:

```javascript
db.employees_v2.createIndex({ dept: 1 });
```

Bulk loading first is faster.

#### Use Bulk Writes

```javascript
db.collection.bulkWrite();
```

Reduces network round trips.

#### Use Read Concern / Write Concern Carefully

Example:

```javascript
{
  writeConcern: {
    w: "majority";
  }
}
```

Ensures durability during migration.

### Common Pitfalls

| Pitfall                    | Impact                     |
| -------------------------- | -------------------------- |
| Migrating without batching | OOM / performance collapse |
| Missing indexes on target  | Slow queries after cutover |
| Large transactions         | Transaction cache pressure |
| No validation step         | Silent data corruption     |
| Full collection scans      | Cluster slowdown           |

### Validation Techniques

```javascript
db.employees.countDocuments();
db.employees_v2.countDocuments();
```

Or checksum validation:

```javascript
db.employees.aggregate([
  {
    $group: {
      _id: null,
      totalSalary: { $sum: "$salary" },
    },
  },
]);
```

Compare totals across collections.

### Follow-up Interview Questions

#### 1. When would you use Change Streams during migration?

Use Change Streams to capture live updates while backfilling large collections, enabling near zero-downtime migrations.

#### 2. Why avoid huge multi-document transactions for migrations?

Large transactions increase memory usage, oplog pressure, and rollback complexity. Batch-based idempotent migrations scale better.
