# Set 7

| S.No. | Question                                                                                              |
| ----- | ----------------------------------------------------------------------------------------------------- |
| 1.    | [How do you count documents in a collection?](#question-1-how-do-you-count-documents-in-a-collection) |

## Question 1. How do you count documents in a collection?

In MongoDB, counting documents means determining how many documents exist in a collection, optionally based on filters. MongoDB provides multiple methods, but the **recommended modern approach is `countDocuments()`**.

## 1. `countDocuments()` (Recommended)

### Definition

Returns the **exact number of documents matching a filter**. It scans based on query criteria and respects indexes.

### Example: Count all employees

```javascript
db.employees.countDocuments({});
```

### Explanation

- `{}` → empty filter means “match all documents”
- Returns total number of documents in `employees` collection

### Example: Count Engineering employees

```javascript
db.employees.countDocuments({ department: "Engineering" });
```

### Explanation

- Filters documents where `department` = "Engineering"
- Uses indexes if available for better performance

## 2. `estimatedDocumentCount()` (Fast, Approximate)

### Definition

Returns an **approximate count** using collection metadata (not scanning documents).

### Example

```javascript
db.employees.estimatedDocumentCount();
```

### Explanation

- No filter allowed
- Reads collection metadata instead of scanning documents
- Very fast but not always 100% accurate during heavy writes/deletes

## 3. Legacy `count()` (Deprecated)

```javascript
db.employees.count({ department: "HR" });
```

Deprecated because:

- Behavior inconsistent in sharded clusters
- Replaced by `countDocuments()` and `estimatedDocumentCount()`

## Performance Considerations

### Use indexes

If filtering:

```javascript
db.employees.createIndex({ department: 1 });
```

- Speeds up `countDocuments({ department: "Engineering" })`

---

### Avoid full collection scan when possible

- `countDocuments()` may scan documents if no index exists
- `estimatedDocumentCount()` avoids scanning but is approximate

## Common Pitfalls

1. **Using `estimatedDocumentCount()` for filtered counts**
   - Not supported
   - Only gives total collection size

2. **Assuming `countDocuments()` is always fast**
   - Without indexes → full scan → slow on large datasets

3. **Using deprecated `count()` in production**
   - Can return inconsistent results in sharded environments

## Real-world Use Case

- Dashboard stats → `estimatedDocumentCount()` (fast)
- Reporting system → `countDocuments()` (accurate)
- Filtered analytics → `countDocuments({ ...query })`

## Quick Summary

| Method                   | Accuracy   | Speed  | Use Case        |
| ------------------------ | ---------- | ------ | --------------- |
| countDocuments()         | Exact      | Medium | Filtered counts |
| estimatedDocumentCount() | Approx     | Fast   | Total count     |
| count()                  | Deprecated | —      | Avoid           |

## Follow-up Interview Questions

### 1. Why is `countDocuments()` preferred over `count()`?

Because it provides accurate results, supports filters properly, and works reliably in sharded clusters.

### 2. How can you optimize counting large collections?

By creating appropriate indexes and avoiding full collection scans during `countDocuments()` queries.
