# Set 12

| S.No. | Question                                                                                                                                                |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between db.collection.update() and replaceOne()?](#question-1-what-is-the-difference-between-dbcollectionupdate-and-replaceone) |

## Question 1. What is the difference between db.collection.update() and replaceOne()?

In MongoDB, **`update()`** (legacy) and **`replaceOne()`** both modify documents, but they behave very differently:

- **`update()`** → modifies _specific fields_ in a document using update operators like `$set`, `$inc`, etc. (or can replace if used without operators in legacy form).
- **`replaceOne()`** → completely **replaces the entire document**, except the `_id`.

## 1. `update()` (Field-level modification)

### Concept

`update()` (or modern `updateOne()` / `updateMany()`) is used to **update specific fields** without touching the rest of the document.

### Example

```javascript
db.employees.update({ name: "Alice" }, { $set: { salary: 90000 } });
```

### Explanation line-by-line

- `{ name: "Alice" }` → filter condition
- `$set: { salary: 90000 }` → only updates salary field
- Other fields (`department`, `skills`, etc.) remain unchanged

### Important behavior

If you forget `$set`:

```javascript
db.employees.update({ name: "Alice" }, { salary: 90000 });
```

This **replaces the entire document** (dangerous legacy behavior).

## 2. `replaceOne()` (Full document replacement)

### Concept

`replaceOne()` **replaces the entire document** matching the filter, except `_id`.

### Example

```javascript
db.employees.replaceOne(
  { name: "Alice" },
  {
    name: "Alice",
    department: "Engineering",
    salary: 95000,
    hireDate: ISODate("2020-01-15"),
    skills: ["JavaScript", "TypeScript"],
  },
);
```

### Explanation

- Filter: `{ name: "Alice" }`
- Entire document is replaced with the new object
- `_id` remains unchanged automatically
- Any missing fields (like `managerId`) are **deleted**

## Key Differences Table

| Feature            | update()             | replaceOne()              |
| ------------------ | -------------------- | ------------------------- |
| Scope              | Partial update       | Full document replacement |
| Operators required | Yes (`$set`, `$inc`) | No operators              |
| Risk of data loss  | Low                  | High (overwrites fields)  |
| \_id handling      | unchanged            | unchanged                 |
| Use case           | Modify few fields    | Redesign entire document  |

## Real-world Use Cases

### Use `update()` when

- Updating salary, status, counters
- Adding/removing fields
- Incrementing values (`$inc`)

Example:

```javascript
db.employees.updateOne({ name: "Bob" }, { $inc: { salary: 5000 } });
```

### Use `replaceOne()` when

- You redesigned schema
- You fetched full object, modified it in app layer, and want to overwrite
- Migrating document structure

## Performance Insight (Interview Tip)

- `update()` is generally **more efficient** because it modifies only required fields.
- `replaceOne()` may cause:
  - higher write cost
  - potential index re-evaluation
  - accidental data loss if fields are omitted

## Common Pitfalls

### ❌ Forgetting `$set`

Leads to accidental full replacement (legacy `update()` behavior).

### ❌ Using replaceOne() with partial data

```javascript
db.employees.replaceOne({ name: "Alice" }, { salary: 100000 });
```

This deletes all other fields → dangerous in production.

## Follow-up Interview Questions

### 1. What is the difference between `updateOne()` and `findOneAndUpdate()`?

👉 `findOneAndUpdate()` returns the modified document; `updateOne()` does not.

### 2. How does MongoDB ensure atomicity in updates?

Single-document updates are atomic by design in MongoDB.
