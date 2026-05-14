# Set 4

| S.No. | Question                                                                                                          |
| ----- | ----------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between save() and insert()?](#question-1-what-is-the-difference-between-save-and-insert) |

## Question 1. What is the difference between save() and insert()?

In MongoDB, `save()` and `insert()` were both historically used to add documents, but there are important differences — especially in modern MongoDB versions.

### 1. Core Difference

| Method     | Purpose                                          | Current Status                    |
| ---------- | ------------------------------------------------ | --------------------------------- |
| `insert()` | Inserts new documents only                       | Deprecated                        |
| `save()`   | Inserts or updates a document depending on `_id` | Removed from modern drivers/shell |

In the latest MongoDB versions, you should use:

- `insertOne()`
- `insertMany()`
- `updateOne()`
- `replaceOne()`

instead of `save()` or `insert()`.

### 2. How `insert()` Worked

`insert()` was used only for inserting new documents.

### Example

```javascript
use interview_db;

db.employees.insert({
  name: "David",
  department: "Finance",
  salary: 70000
});
```

### What Happens?

- MongoDB inserts a new document.
- If `_id` is not provided, MongoDB auto-generates it.
- If the same `_id` already exists, insertion fails with a duplicate key error.

### 3. How `save()` Worked

`save()` behaved like an **upsert** operation.

### Behavior

- If `_id` does NOT exist → insert new document
- If `_id` exists → replace existing document

### Example 1 — Insert Using `save()`

```javascript
db.employees.save({
  name: "Emma",
  department: "HR",
  salary: 65000,
});
```

### Result

- Since `_id` is absent, MongoDB inserts a new document.

### Example 2 — Update Using `save()`

```javascript
var emp = db.employees.findOne({ name: "Alice" });

emp.salary = 90000;

db.employees.save(emp);
```

### Line-by-Line Explanation

### Step 1

```javascript
var emp = db.employees.findOne({ name: "Alice" });
```

- Fetches Alice’s document including `_id`.

### Step 2

```javascript
emp.salary = 90000;
```

- Modifies the in-memory document.

### Step 3

```javascript
db.employees.save(emp);
```

- Since `_id` exists, MongoDB replaces the entire document.

### 4. Major Pitfall of `save()`

`save()` replaces the whole document.

Example:

```javascript
db.employees.save({
  _id: ObjectId("..."),
  name: "Alice",
});
```

This removes fields like:

- `department`
- `salary`
- `skills`

because the entire document gets overwritten.

That’s one major reason MongoDB moved away from `save()`.

### 5. Modern Recommended Approach

Instead of `save()`:

```javascript
db.employees.updateOne(
  { name: "Alice" },
  {
    $set: {
      salary: 90000,
    },
  },
);
```

### Why Better?

- Updates only specific fields
- Safer
- More efficient
- Prevents accidental document replacement

### 6. Performance Considerations

### `insert()`

- Fast for pure inserts
- No read-before-write needed

### `save()`

- Can involve replacement logic
- More dangerous for large documents
- May overwrite unintended fields

Modern APIs (`insertOne`, `updateOne`) are optimized and clearer.

### 7. Interview Summary

- `insert()` only inserts new documents.
- `save()` inserts or replaces documents based on `_id`.
- Both are legacy methods.
- Modern MongoDB applications should use:
  - `insertOne()`
  - `insertMany()`
  - `updateOne()`
  - `replaceOne()`

### 8. Common Follow-Up Interview Questions

### Q1: Why was `save()` deprecated?

Because it could unintentionally replace entire documents and caused ambiguity between insert and update operations.

### Q2: What is the modern equivalent of `save()`?

- `replaceOne()` → for full replacement
- `updateOne()` with `$set` → for partial updates
- `upsert: true` → if insert-or-update behavior is needed

Example:

```javascript
db.employees.updateOne(
  { name: "Alice" },
  {
    $set: { salary: 90000 },
  },
  {
    upsert: true,
  },
);
```
