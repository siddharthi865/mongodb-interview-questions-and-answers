# Set 16

| S.No. | Question                                                                                                        |
| ----- | --------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you perform a left outer join in MongoDB?](#question-1-how-do-you-perform-a-left-outer-join-in-mongodb) |

## Question 1. How do you perform a left outer join in MongoDB?

In MongoDB, a **left outer join** is performed using the aggregation framework’s `$lookup` stage. It works similarly to SQL’s `LEFT OUTER JOIN` — all documents from the **left collection** are returned, and matching documents from the **right collection** are added as an array.

MongoDB introduced `$lookup` to support joins across collections while keeping MongoDB’s document-oriented design.

## Example Scenario

Suppose we have:

### `employees` collection

```javascript
use interview_db;

db.employees.insertMany([
  {
    _id: 1,
    name: "Alice",
    department: "Engineering"
  },
  {
    _id: 2,
    name: "Bob",
    department: "Engineering"
  },
  {
    _id: 3,
    name: "Charlie",
    department: "HR"
  }
]);
```

### `departments` collection

```javascript
db.departments.insertMany([
  {
    _id: "ENG",
    name: "Engineering",
    location: "Building A",
  },
  {
    _id: "HR",
    name: "HR",
    location: "Building B",
  },
]);
```

## Performing a Left Outer Join with `$lookup`

```javascript
db.employees.aggregate([
  {
    $lookup: {
      from: "departments",
      localField: "department",
      foreignField: "name",
      as: "departmentDetails",
    },
  },
]);
```

## Line-by-Line Explanation

## 1. `$lookup`

```javascript
{
  $lookup: {
```

- Performs the join operation.
- Equivalent to SQL JOIN logic.

## 2. `from`

```javascript
from: "departments";
```

- Target collection to join with.

Equivalent SQL:

```sql
JOIN departments
```

## 3. `localField`

```javascript
localField: "department";
```

- Field from the current (`employees`) collection.

## 4. `foreignField`

```javascript
foreignField: "name";
```

- Matching field in the `departments` collection.

## 5. `as`

```javascript
as: "departmentDetails";
```

- Output array field containing matched documents.

## Sample Output

```javascript
[
  {
    _id: 1,
    name: "Alice",
    department: "Engineering",
    departmentDetails: [
      {
        _id: "ENG",
        name: "Engineering",
        location: "Building A",
      },
    ],
  },
];
```

Notice:

- Even if no department matches, the employee document still appears.
- That’s why it behaves as a **left outer join**.

## Converting Array Result into Object

Since `$lookup` always returns an array, we often use `$unwind`.

```javascript
db.employees.aggregate([
  {
    $lookup: {
      from: "departments",
      localField: "department",
      foreignField: "name",
      as: "departmentDetails",
    },
  },
  {
    $unwind: {
      path: "$departmentDetails",
      preserveNullAndEmptyArrays: true,
    },
  },
]);
```

## Why `preserveNullAndEmptyArrays`?

This preserves unmatched employees, maintaining left outer join behavior.

Without it, documents with no match are dropped.

## Advanced `$lookup` with Pipeline

Modern MongoDB versions support pipeline-based joins:

```javascript
db.employees.aggregate([
  {
    $lookup: {
      from: "departments",
      let: { deptName: "$department" },
      pipeline: [
        {
          $match: {
            $expr: {
              $eq: ["$name", "$$deptName"],
            },
          },
        },
      ],
      as: "departmentDetails",
    },
  },
]);
```

This is useful for:

- Complex conditions
- Multiple join filters
- Aggregations inside joins
- Date comparisons
- Correlated subqueries

## Performance Considerations

## 1. Index the Foreign Field

Create an index on the joined field:

```javascript
db.departments.createIndex({ name: 1 });
```

Without indexing, `$lookup` can become expensive on large collections.

## 2. Avoid Huge Arrays

`$lookup` loads matching documents into memory.

Large joins may:

- Increase RAM usage
- Cause slower aggregation performance

## 3. Use Pipeline Filtering Early

Reduce joined data:

```javascript
pipeline: [
  {
    $match: {
      active: true,
    },
  },
];
```

## SQL Comparison

| SQL             | MongoDB                       |
| --------------- | ----------------------------- |
| LEFT OUTER JOIN | `$lookup`                     |
| ON condition    | `localField` + `foreignField` |
| Joined table    | `from`                        |
| Result alias    | `as`                          |

## Common Interview Follow-up Questions

## 1. Does MongoDB support joins across databases?

No. `$lookup` works only within the same database.

## 2. What is the difference between embedding and `$lookup`?

- **Embedding** → Faster reads, denormalized data
- **$lookup** → Better normalization, avoids duplication

Embedding is preferred when related data is frequently accessed together.
