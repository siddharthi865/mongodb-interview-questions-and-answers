# Set 17

| S.No. | Question                                                                                                                        |
| ----- | ------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you query for documents where a field is missing?](#question-1-how-do-you-query-for-documents-where-a-field-is-missing) |

## Question 1. How do you query for documents where a field is missing?

In MongoDB, you can query documents where a field is **missing** using the `$exists` operator.

`$exists: false` matches documents that **do not contain** the specified field at all.

## Basic Concept

MongoDB documents are schema-flexible, so not every document must contain the same fields.

Example:

```javascript
{
  name: "Alice",
  department: "Engineering"
}
```

```javascript
{
  name: "Bob",
  department: "Engineering",
  managerId: ObjectId("...")
}
```

If you want to find employees without a `managerId`, you query for documents where the field does not exist.

## Example Using Sample Dataset

```javascript
use interview_db;

// Find employees where managerId field is missing
db.employees.find({
  managerId: { $exists: false }
});
```

## Line-by-Line Explanation

## 1. `$exists`

```javascript
managerId: {
  $exists: false;
}
```

- Checks whether the field exists in the document.
- `false` means:
  - Return documents where `managerId` is absent.

## Example Matching Documents

Suppose collection contains:

```javascript
[
  {
    name: "Alice",
    department: "Engineering",
  },
  {
    name: "Bob",
    department: "Engineering",
    managerId: ObjectId("..."),
  },
];
```

Query result:

```javascript
{
  name: "Alice",
  department: "Engineering"
}
```

Because `managerId` does not exist.

## Difference Between Missing and Null

This is a very common MongoDB interview topic.

## Field Missing

```javascript
{
  name: "Alice";
}
```

## Field Present but Null

```javascript
{
  name: "Bob",
  managerId: null
}
```

These are different.

## Query Only Missing Fields

```javascript
db.employees.find({
  managerId: { $exists: false },
});
```

Matches only documents where field is absent.

## Query Null OR Missing

```javascript
db.employees.find({
  managerId: null,
});
```

Matches:

- `managerId: null`
- missing `managerId`

This behavior is important in interviews.

## Query Only Explicit Null Values

```javascript
db.employees.find({
  managerId: {
    $exists: true,
    $eq: null,
  },
});
```

This returns documents where:

- field exists
- value is explicitly `null`

## Performance Considerations

## Index Usage

`$exists` can use indexes efficiently when the field is indexed.

Example:

```javascript
db.employees.createIndex({ managerId: 1 });
```

Queries with:

```javascript
{
  managerId: {
    $exists: true;
  }
}
```

perform better with an index.

However:

```javascript
{
  managerId: {
    $exists: false;
  }
}
```

may still require more scanning because MongoDB must identify missing fields.

## Real-World Use Cases

- Finding incomplete profiles
- Detecting missing metadata
- Data cleanup/migration tasks
- Identifying documents needing enrichment
- Validation auditing

Example:

```javascript
db.orders.find({
  trackingNumber: { $exists: false },
});
```

Finds orders not yet shipped.

## Common Pitfalls

## 1. Confusing `null` with missing

```javascript
{
  field: null;
}
```

matches both.

Use `$exists` for precise control.

## 2. Assuming all documents share same schema

MongoDB is schema-flexible, so missing fields are common.

## Follow-Up Interview Questions

## 1. Can `$exists` use an index?

Yes. Queries with `$exists: true` benefit more directly from indexes. `$exists: false` may still require collection scanning.

## 2. How do you validate required fields in MongoDB?

Using schema validation:

```javascript
db.createCollection("employees", {
  validator: {
    $jsonSchema: {
      required: ["name", "department"],
    },
  },
});
```

This prevents missing required fields in future documents.
