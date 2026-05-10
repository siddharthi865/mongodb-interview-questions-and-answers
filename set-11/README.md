# Set 11

| S.No. | Question                                                                                  |
| ----- | ----------------------------------------------------------------------------------------- |
| 1.    | [What are the key features of MongoDB?](#question-1-what-are-the-key-features-of-mongodb) |

## Question 1. What are the key features of MongoDB?

### 1. Document-Oriented Storage (Flexible Schema)

MongoDB stores data in **BSON documents (Binary JSON)** instead of rows and columns. Each document can have a different structure.

**Why it matters:**

- No rigid schema like SQL
- Easy evolution of application data models
- Ideal for agile development

**Example:**

```javascript
db.employees.insertOne({
  name: "Alice",
  department: "Engineering",
  skills: ["JavaScript", "React"],
});
```

Another document in same collection can have extra fields:

```javascript
{
  name: "Bob",
  salary: 80000,
  location: "Gurgaon"
}
```

### 2. Horizontal Scalability (Sharding)

MongoDB supports **automatic data distribution across multiple servers (shards)**.

**How it works:**

- Data is split using a **shard key**
- Each shard holds a portion of data
- Enables scaling to billions of documents

**Example concept:**

```javascript
sh.shardCollection("company.employees", { department: 1 });
```

**Use case:**
Large-scale applications like e-commerce catalogs or logging systems.

### 3. High Availability (Replica Sets)

MongoDB uses **replica sets** for redundancy.

**Structure:**

- Primary node → handles writes
- Secondary nodes → replicate data
- Automatic failover if primary fails

**Benefit:**

- No downtime
- Data durability

### 4. Rich Query Language

MongoDB supports powerful queries including:

- Filtering
- Sorting
- Aggregation
- Text search
- Geospatial queries

**Example:**

```javascript
db.employees.find({ department: "Engineering", salary: { $gt: 70000 } });
```

### 5. Aggregation Framework

Used for data processing pipelines (similar to SQL GROUP BY but more powerful).

**Example:**

```javascript
db.employees.aggregate([
  { $match: { department: "Engineering" } },
  { $group: { _id: "$department", avgSalary: { $avg: "$salary" } } },
]);
```

### 6. Indexing Support

MongoDB supports multiple index types:

- Single field index
- Compound index
- Text index
- Geospatial index
- TTL index

**Example:**

```javascript
db.employees.createIndex({ department: 1, salary: -1 });
```

**Benefit:**

- Faster queries
- Optimized performance

### 7. Transactions (ACID Compliance)

MongoDB supports **multi-document ACID transactions** (since v4.0+).

**Example:**

```javascript
const session = db.getMongo().startSession();
session.startTransaction();

session
  .getDatabase("interview_db")
  .employees.updateOne({ name: "Alice" }, { $set: { salary: 90000 } });

session.commitTransaction();
```

### 8. Built-in Replication & Fault Tolerance

- Automatic replication of data
- Self-healing cluster
- No single point of failure

### 9. Change Streams (Real-Time Data)

Allows applications to listen to database changes in real time.

**Use case:**

- Notification systems
- Real-time dashboards

```javascript
db.employees.watch().on("change", (data) => {
  printjson(data);
});
```

### 10. Cloud & Atlas Integration

MongoDB Atlas provides:

- Managed cloud database
- Auto scaling
- Backup & monitoring
- Global clusters

## Common Interview Follow-ups

### 1. How is MongoDB different from SQL databases?

MongoDB is schema-flexible and document-based, while SQL databases are structured and table-based with fixed schemas.

### 2. What are the disadvantages of MongoDB?

- Joins are limited compared to SQL
- Memory usage can be higher
- Requires careful indexing for performance
