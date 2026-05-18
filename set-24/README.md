# Set 24

| S.No. | Question                                                                                                            |
| ----- | ------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you implement bulk writes with retry logic?](#question-1-how-do-you-implement-bulk-writes-with-retry-logic) |

## Question 1. How do you implement bulk writes with retry logic?

Bulk writes in MongoDB allow you to execute multiple write operations (`insert`, `update`, `delete`, `replace`) in a single request. Adding **retry logic** is important in production systems to handle transient failures such as:

- Network interruptions
- Primary stepdowns in replica sets
- Temporary write conflicts
- Shard migrations in sharded clusters

MongoDB drivers support **Retryable Writes** automatically for many single-document operations, but for `bulkWrite()`, you often implement custom retry handling at the application level.

### 1. Basic `bulkWrite()` Example

```javascript
use interview_db;

const operations = [
  {
    insertOne: {
      document: {
        name: "David",
        department: "Engineering",
        salary: 85000
      }
    }
  },
  {
    updateOne: {
      filter: { name: "Alice" },
      update: { $set: { salary: 78000 } }
    }
  },
  {
    deleteOne: {
      filter: { name: "Charlie" }
    }
  }
];

db.employees.bulkWrite(operations);
```

#### Explanation

- `insertOne` → inserts a new employee
- `updateOne` → updates Alice’s salary
- `deleteOne` → removes Charlie
- All operations are sent in a single network round-trip

### 2. Implementing Retry Logic

In production, wrap the bulk operation in retry handling.

```javascript
function executeBulkWriteWithRetry(collection, operations, maxRetries = 3) {
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      const result = collection.bulkWrite(operations, {
        ordered: false,
        writeConcern: { w: "majority" },
      });

      print("Bulk write successful");
      return result;
    } catch (error) {
      attempt++;

      print(`Attempt ${attempt} failed`);

      // Retry only transient/retryable errors
      if (
        error.hasErrorLabel &&
        (error.hasErrorLabel("RetryableWriteError") ||
          error.hasErrorLabel("TransientTransactionError"))
      ) {
        print("Retrying bulk operation...");

        sleep(1000 * attempt); // exponential backoff
      } else {
        throw error;
      }
    }
  }

  throw new Error("Bulk write failed after retries");
}
```

#### Usage

```javascript
executeBulkWriteWithRetry(db.employees, operations);
```

### Key Concepts

#### Ordered vs Unordered

##### Ordered Bulk Write

```javascript
{
  ordered: true;
}
```

- Executes sequentially
- Stops on first failure
- Useful when operation order matters

##### Unordered Bulk Write

```javascript
{
  ordered: false;
}
```

- Executes in parallel internally
- Continues even if some operations fail
- Better throughput for large workloads

For retry systems, `ordered: false` is usually preferred.

### Performance Optimizations

#### Use Batch Sizes

Very large bulk operations consume memory and network bandwidth.

Recommended approach:

```javascript
const batchSize = 1000;
```

Process documents in chunks.

#### Use Majority Write Concern Carefully

```javascript
writeConcern: {
  w: "majority";
}
```

Provides durability but increases latency.

For logging systems or analytics:

```javascript
writeConcern: {
  w: 1;
}
```

may be sufficient.

#### Ensure Idempotency

Retries can accidentally duplicate writes.

Safer patterns:

```javascript
updateOne: {
  filter: { _id: employeeId },
  update: { $set: data },
  upsert: true
}
```

Avoid non-idempotent updates like:

```javascript
$inc: {
  counter: 1;
}
```

unless duplicate increments are acceptable.

### Common Pitfalls

#### Duplicate Inserts on Retry

If retry occurs after partial success:

```javascript
insertOne;
```

may insert duplicates.

Solution:

- Use unique indexes
- Use deterministic `_id` values

Example:

```javascript
{
  _id: "EMP_1001";
}
```

#### Partial Failures

With unordered bulk writes:

- Some operations may succeed
- Others may fail

Always inspect:

```javascript
result.getWriteErrors();
```

### Real-World Use Cases

- ETL pipelines
- Kafka/MongoDB consumers
- Inventory synchronization
- Event ingestion systems
- High-volume analytics imports

### Follow-up Interview Questions

#### 1. Does MongoDB automatically retry `bulkWrite()` operations?

No. MongoDB drivers automatically retry many single-document writes, but complex `bulkWrite()` retry handling is typically implemented manually.

#### 2. Why is idempotency important in retry systems?

Because retries may execute the same operation multiple times. Idempotent operations ensure repeated execution produces the same final state.
