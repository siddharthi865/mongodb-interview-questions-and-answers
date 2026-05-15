# Set 6

## Question 1. What are the advantages of using MongoDB over SQL databases?

> MongoDB is a **NoSQL document-oriented database** that stores data in flexible JSON-like documents (BSON). Unlike traditional SQL databases that use rigid tables and schemas, MongoDB offers **flexibility, scalability, and high performance for modern applications**.

### Key Advantages

#### 1. Flexible Schema (Schema-less Design)

MongoDB does not require a fixed schema like SQL databases.

##### Example

```javascript
db.employees.insertOne({
  name: "Alice",
  department: "Engineering",
  skills: ["JavaScript", "React"],
});

db.employees.insertOne({
  name: "Bob",
  department: "Engineering",
  salary: 80000,
  isRemote: true,
});
```

##### Explanation

- Documents in the same collection can have different fields.
- No need for ALTER TABLE when requirements change.
- Ideal for agile development.

#### 2. High Performance for Read/Write Operations

- Embedded documents reduce JOIN operations.
- Data is stored in a way that closely matches application objects.

##### SQL vs MongoDB:

- SQL: Requires multiple JOINs
- MongoDB: Single document fetch

Example:

```javascript
db.employees.find({ department: "Engineering" });
```

#### 3. Horizontal Scalability (Sharding)

MongoDB scales out easily using **sharding**.

###### Key concept:

- Data is distributed across multiple servers.
- Handles large-scale, high-traffic applications.

Example use case:

- E-commerce platforms
- Social media apps

#### 4. Embedded Data Model (Faster Access)

Instead of joins, MongoDB allows embedding related data.

##### Example

```javascript
{
  name: "Alice",
  department: "Engineering",
  projects: [
    { name: "Project A", status: "Active" },
    { name: "Project B", status: "Completed" }
  ]
}
```

##### Benefit

- Faster reads
- Fewer database queries

#### 5. High Availability (Replica Sets)

MongoDB provides automatic replication.

- Primary node handles writes
- Secondary nodes replicate data
- Automatic failover ensures uptime

#### 6. Developer-Friendly (JSON-like Structure)

- Natural fit for JavaScript/Node.js apps
- Easier mapping between backend and database objects

#### 7. Powerful Query & Aggregation Framework

MongoDB supports advanced analytics using aggregation pipelines.

```javascript
db.employees.aggregate([
  { $match: { department: "Engineering" } },
  { $group: { _id: null, avgSalary: { $avg: "$salary" } } },
]);
```

#### 8. Cloud-Native (MongoDB Atlas)

- Fully managed cloud database
- Built-in backups, monitoring, scaling

### When SQL Might Be Better (Important Interview Insight)

MongoDB is NOT always better:

- Complex transactions across multiple tables → SQL is stronger
- Strong relational integrity → SQL preferred

### Summary

MongoDB is preferred when you need:

- Flexibility (dynamic schema)
- High scalability
- Fast development cycles
- Large-scale distributed systems

### Follow-up Interview Questions

#### 1. When should you NOT use MongoDB?

When you need complex joins, strict ACID compliance across many tables, or highly relational data.

#### 2. How does MongoDB achieve horizontal scaling?

Through **sharding**, where data is partitioned across multiple servers using a shard key.

If you want, I can also give a **MongoDB vs MySQL comparison table (very common interview question)** or real-world architecture examples.
