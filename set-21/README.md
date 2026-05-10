# Set 21

| S.No. | Question                                                                                                      |
| ----- | ------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you perform a distinct query in MongoDB?](#question-1-how-do-you-perform-a-distinct-query-in-mongodb) |

## Question 1. How do you perform a distinct query in MongoDB?

A **distinct query** in MongoDB is used to retrieve all unique values for a specific field across documents in a collection. It is similar to SQL’s `SELECT DISTINCT`.

MongoDB provides the `distinct()` method:

```javascript
db.collection.distinct(field, query);
```

- `field` → The field for which unique values are required.
- `query` → Optional filter to limit documents before extracting distinct values.

### Example Using Sample Dataset

```javascript
use interview_db;

// Get unique departments
db.employees.distinct("department");
```

#### Output

```javascript
["Engineering", "HR"];
```

#### Explanation

- MongoDB scans the `department` field in all documents.
- Duplicate values are removed.
- Only unique department names are returned.

### Distinct Query with Filter

Suppose you want unique skills only for Engineering employees.

```javascript
db.employees.distinct("skills", { department: "Engineering" });
```

#### Output

```javascript
["JavaScript", "React", "Node.js", "MongoDB"];
```

#### Line-by-Line Explanation

```javascript
db.employees.distinct(
```

- Calls the distinct operation on the `employees` collection.

```javascript
"skills",
```

- Extracts unique values from the `skills` array field.
- MongoDB automatically flattens arrays during distinct operations.

```javascript
{
  department: "Engineering";
}
```

- Filters documents before applying distinct.

### Important MongoDB Behaviors

#### 1. Works on Array Fields

If the field contains arrays, MongoDB returns unique array elements.

Example:

```javascript
db.employees.distinct("skills");
```

Result:

```javascript
["JavaScript", "React", "Node.js", "MongoDB", "Recruitment"];
```

### Performance Optimization

#### Create an Index

Distinct queries perform much better with indexes.

```javascript
db.employees.createIndex({ department: 1 });
```

For filtered distinct queries:

```javascript
db.employees.createIndex({
  department: 1,
  skills: 1,
});
```

#### Why?

- MongoDB can use the index to avoid full collection scans.
- Reduces memory and disk I/O.

### Distinct vs Aggregation

You can also achieve distinct behavior using aggregation:

```javascript
db.employees.aggregate([
  {
    $group: {
      _id: "$department",
    },
  },
]);
```

#### When to Use Which?

| Method       | Best For                                  |
| ------------ | ----------------------------------------- |
| `distinct()` | Simple unique value retrieval             |
| `$group`     | Advanced transformations, counts, sorting |

### Common Pitfalls

#### 1. Large Result Sets

`distinct()` returns all unique values in memory.
Very high-cardinality fields may consume significant RAM.

#### 2. Sharded Clusters

In sharded environments, MongoDB gathers distinct values from all shards, which can increase network overhead.

### Interview Follow-Up Questions

#### 1. Can `distinct()` use indexes?

Yes. If an appropriate index exists on the distinct field (and query filter fields), MongoDB can optimize the operation significantly.

#### 2. What is the difference between `distinct()` and `$group`?

- `distinct()` only returns unique values.
- `$group` supports aggregations like counts, sums, averages, sorting, and additional pipeline stages.
