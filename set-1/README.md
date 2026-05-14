# Set 1

| S.No. | Question                                                                                                                |
| ----- | ----------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is MongoDB?](#question-1-what-is-mongodb)                                                                         |
| 2.    | [How is MongoDB different from a relational database?](#question-2-how-is-mongodb-different-from-a-relational-database) |

## Question 1. What is MongoDB?

[MongoDB Official Documentation](https://www.mongodb.com/docs/manual/introduction/)

> MongoDB is a **NoSQL, document-oriented database** designed for high performance, scalability, and flexible schema design. Instead of storing data in rows and columns like traditional relational databases (MySQL, PostgreSQL), MongoDB stores data as **BSON documents** (Binary JSON format).

A MongoDB database contains:

- **Database** → Similar to a schema/database in SQL
- **Collection** → Similar to a table
- **Document** → Similar to a row, but schema-flexible

### Core Concept

Example document in MongoDB:

```javascript
{
  name: "Alice",
  department: "Engineering",
  salary: 75000,
  skills: ["JavaScript", "React"]
}
```

Unlike relational databases:

- Documents in the same collection can have different fields
- Nested objects and arrays are supported naturally
- Schema can evolve without migrations

### Why MongoDB is Popular

### 1. Flexible Schema

MongoDB allows dynamic document structures.

Example:

```javascript
{
  name: "Bob",
  department: "Engineering"
}
```

Another document in the same collection:

```javascript
{
  name: "Charlie",
  department: "HR",
  certifications: ["HR Expert"]
}
```

No ALTER TABLE is required.

### 2. High Performance

MongoDB supports:

- Indexing
- In-memory processing
- Aggregation pipelines
- Horizontal scaling using sharding

### 3. Rich Query Language

MongoDB supports:

- Filtering
- Sorting
- Aggregations
- Joins using `$lookup`
- Transactions
- Full-text search
- Geospatial queries

### Example Using MongoDB Shell

```javascript
use interview_db;

// Insert sample documents
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

### Explanation

```javascript
use interview_db;
```

- Switches to (or creates) the database `interview_db`

```javascript
db.employees.insertMany([...])
```

- `employees` → Collection name
- `insertMany()` → Inserts multiple documents at once

### Query Example

```javascript
db.employees.find({ department: "Engineering" });
```

### What Happens Here?

- `find()` retrieves documents
- `{ department: "Engineering" }` is the filter condition
- MongoDB automatically searches matching documents

### MongoDB Architecture Highlights

### BSON Format

MongoDB internally stores data as BSON, which supports:

- Date
- Binary data
- ObjectId
- Decimal128
- Arrays
- Embedded documents

### Replication

MongoDB provides **Replica Sets** for high availability.

- Primary node handles writes
- Secondary nodes replicate data
- Automatic failover supported

### Sharding

MongoDB supports horizontal scaling through sharding.

Data is distributed across multiple servers automatically.

### Real-World Use Cases

MongoDB is commonly used for:

- E-commerce catalogs
- Real-time analytics
- Content management systems
- IoT applications
- Social media platforms
- Event-driven microservices

### Performance Optimization Tip

Create indexes for frequently queried fields:

```javascript
db.employees.createIndex({ department: 1 });
```

This improves query performance significantly for department-based searches.

### Common Interview Follow-up Questions

### 1. What is the difference between MongoDB and SQL databases?

| MongoDB                   | SQL Database            |
| ------------------------- | ----------------------- |
| Document-oriented         | Table-oriented          |
| Flexible schema           | Fixed schema            |
| BSON documents            | Rows & columns          |
| Horizontal scaling easier | Vertical scaling common |

### 2. What is BSON?

BSON (Binary JSON) is MongoDB’s internal storage format that extends JSON with additional data types like Date, ObjectId, and Binary data.

## Question 2. How is MongoDB different from a relational database?

[MongoDB Data Model Documentation](https://www.mongodb.com/docs/manual/core/data-modeling-introduction)

MongoDB and relational databases solve the same core problem—storing and retrieving data—but they use very different architectures and data models.

### Key Differences

| Feature        | MongoDB                            | Relational Database (MySQL/PostgreSQL) |
| -------------- | ---------------------------------- | -------------------------------------- |
| Data Model     | Document-oriented                  | Table-oriented                         |
| Storage Format | BSON Documents                     | Rows & Columns                         |
| Schema         | Flexible / Dynamic                 | Fixed / Predefined                     |
| Relationships  | Embedded documents or references   | Foreign keys & joins                   |
| Scaling        | Horizontal scaling (Sharding)      | Primarily vertical scaling             |
| Transactions   | Supported                          | Supported                              |
| Query Language | MongoDB Query Language (MQL)       | SQL                                    |
| Joins          | `$lookup`                          | Native JOIN                            |
| Best For       | Rapidly changing/unstructured data | Highly structured data                 |

### 1. Data Storage Model

### Relational Database

Data is stored in tables.

### Example

### Employees Table

| id  | name  | department  |
| --- | ----- | ----------- |
| 1   | Alice | Engineering |

### Skills Table

| emp_id | skill |
| ------ | ----- |
| 1      | React |

Data is normalized across multiple tables.

### MongoDB

Data is stored as documents.

```javascript
{
  name: "Alice",
  department: "Engineering",
  skills: ["JavaScript", "React"]
}
```

MongoDB promotes **denormalization** when beneficial for performance.

### 2. Schema Flexibility

### Relational DB

Schema must be predefined.

```sql
CREATE TABLE employees (
  id INT,
  name VARCHAR(50),
  department VARCHAR(50)
);
```

Adding a new column requires schema alteration.

### MongoDB

Documents can have different structures in the same collection.

```javascript
{
  name: "Bob",
  department: "Engineering"
}
```

```javascript
{
  name: "Charlie",
  department: "HR",
  certifications: ["HR Expert"]
}
```

No migration required.

### 3. Relationships

### Relational DB → JOINs

```sql
SELECT e.name, s.skill
FROM employees e
JOIN skills s ON e.id = s.emp_id;
```

### MongoDB → Embedding or References

### Embedded Model

```javascript
{
  name: "Alice",
  skills: ["JavaScript", "React"]
}
```

### Reference Model

```javascript
{
  name: "Bob",
  managerId: ObjectId("...")
}
```

MongoDB also supports joins using aggregation `$lookup`.

### MongoDB Example

Using your sample dataset:

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

### Query Comparison

### SQL Query

```sql
SELECT * FROM employees
WHERE department = 'Engineering';
```

### MongoDB Query

```javascript
db.employees.find({
  department: "Engineering",
});
```

### Explanation

```javascript
find();
```

- Retrieves documents from collection

```javascript
{
  department: "Engineering";
}
```

- Filter condition
- Similar to SQL `WHERE`

### 4. Scalability

### Relational DB

Traditionally scales vertically:

- More CPU
- More RAM
- Bigger server

### MongoDB

Designed for horizontal scaling using **Sharding**.

MongoDB distributes data across multiple servers automatically.

[MongoDB Sharding Documentation](https://www.mongodb.com/docs/manual/sharding)

### 5. Transactions

Earlier MongoDB versions had limited transaction support, but modern MongoDB supports:

- Multi-document ACID transactions
- Replica set transactions
- Distributed transactions

However, MongoDB performance is best when data is modeled to minimize cross-document transactions.

### When to Use MongoDB vs RDBMS

## MongoDB is Ideal For

- Rapid development
- Evolving schemas
- JSON-heavy APIs
- Event-driven systems
- Real-time analytics
- Large-scale distributed systems

### Relational DB is Ideal For

- Complex JOIN-heavy queries
- Strong relational consistency
- Highly structured transactional systems
- Financial systems

### Performance Consideration

MongoDB performs best when related data is embedded appropriately because:

- Fewer joins
- Faster reads
- Better locality of data

Example optimization:

```javascript
db.employees.createIndex({ department: 1 });
```

Creates an index for faster department searches.

### Common Interview Follow-up Questions

### 1. Does MongoDB support JOIN operations?

Yes. MongoDB supports joins using the aggregation stage `$lookup`, though embedding is often preferred for performance.

### 2. Is MongoDB schema-less?

Not completely. MongoDB has a flexible schema, but schema validation can be enforced using validators and JSON Schema.
