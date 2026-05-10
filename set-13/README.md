# Set 13

| S.No. | Question                                                                                             |
| ----- | ---------------------------------------------------------------------------------------------------- |
| 1.    | [How do you use $arrayElemAt in aggregation?](#question-1-how-do-you-use-arrayelemat-in-aggregation) |

## Question 1. How do you use $arrayElemAt in aggregation?

### What is `$arrayElemAt` in MongoDB Aggregation?

`$arrayElemAt` is an aggregation operator used to **retrieve an element from an array at a specific index**. It is especially useful when you want to extract a single value from an array field without unwinding it.

Syntax:

```javascript
{ $arrayElemAt: [ <array>, <index> ] }
```

- `<array>` → the array field or expression
- `<index>` → position of the element (0-based indexing)
  - `0` → first element
  - `1` → second element
  - `-1` → last element

## Use Case with Sample Dataset

Let’s enhance the sample collection:

```javascript
db.employees.updateMany(
  {},
  {
    $set: {
      previousCompanies: ["Google", "Microsoft", "Amazon"],
    },
  },
);
```

## Example 1: Get First Skill of Each Employee

```javascript
db.employees.aggregate([
  {
    $project: {
      name: 1,
      firstSkill: { $arrayElemAt: ["$skills", 0] },
    },
  },
]);
```

### Explanation

- `$project` → reshapes output documents
- `$skills` → array field
- `0` → fetches first element of array
- Returns:

  ```json
  { "name": "Alice", "firstSkill": "JavaScript" }
  ```

## Example 2: Get Last Previous Company

```javascript
db.employees.aggregate([
  {
    $project: {
      name: 1,
      lastCompany: { $arrayElemAt: ["$previousCompanies", -1] },
    },
  },
]);
```

### Explanation:

- `-1` index → last element of array
- Useful when array length is unknown

## Example 3: Combine with `$split` (Advanced Pattern)

Suppose we want domain from email:

```javascript
db.employees.aggregate([
  {
    $addFields: {
      emailParts: { $split: ["alice@company.com", "@"] },
    },
  },
  {
    $project: {
      domain: { $arrayElemAt: ["$emailParts", 1] },
    },
  },
]);
```

### Explanation

- `$split` creates array: `["alice", "company.com"]`
- `$arrayElemAt` extracts `"company.com"`

## Performance Notes & Pitfalls

### Efficient when

- Used inside `$project` or `$addFields`
- No need for `$unwind` (which is more expensive)

### Common pitfalls

- Index out of range → returns `null`
- Applying on non-array fields → error or unexpected result
- Misunderstanding 0-based indexing

## Real-world Use Cases

- Fetching:
  - first login event
  - last transaction
  - top-rated item

- Extracting structured values from:
  - `$split`
  - `$map`
  - nested arrays

- Building APIs where only one array element is needed

## Related Interview Follow-ups

### 1. When would you use `$arrayElemAt` instead of `$unwind`?

Use `$arrayElemAt` when you only need a **single element**; use `$unwind` when you need to **process all elements individually**.

### 2. What happens if the index is out of range?

MongoDB returns `null` without throwing an error, which is useful for safe projections.
