# Set 8

| S.No. | Question                                                                                                      |
| ----- | ------------------------------------------------------------------------------------------------------------- |
| 1.    | [Explain the $text operator for full-text search](#question-1-explain-the-text-operator-for-full-text-search) |

## Question 1. Explain the $text operator for full-text search

## `$text` Operator in MongoDB (Full-Text Search)

### Core Concept (Interview Definition)

The **`$text` operator** in MongoDB is used to perform **full-text search queries on string content** stored in documents. It allows you to search for words or phrases within fields that are part of a **text index**, supporting features like stemming, case-insensitive search, and relevance scoring.

In simple terms:
It enables “Google-like search” inside MongoDB collections.

## Step 1: Create a Text Index (Mandatory)

Before using `$text`, you must create a **text index** on one or more string fields.

### Example: Create text index on `name` and `skills`

```javascript
db.employees.createIndex({
  name: "text",
  skills: "text",
});
```

### Explanation

- `"text"` tells MongoDB to build a full-text index
- Multiple fields can be included, but only **one text index per collection is allowed**
- MongoDB automatically applies:
  - stemming (e.g., "running" → "run")
  - case-insensitive search
  - stop-word filtering (e.g., "the", "and")

Docs reference: MongoDB Text Indexes → [https://www.mongodb.com/docs/manual/core/index-text/](https://www.mongodb.com/docs/manual/core/index-text/)

## Step 2: Using `$text` Query Operator

### Example: Search employees with skill "MongoDB"

```javascript
db.employees.find({
  $text: { $search: "MongoDB" },
});
```

### Explanation line-by-line

- `$text` → activates full-text search engine
- `$search: "MongoDB"` → keyword(s) to search
- MongoDB scans **text index only (not full collection scan)**

## Step 3: Relevance Scoring (`$meta`)

MongoDB assigns a **text score** based on relevance.

### Example: Return sorted results by relevance

```javascript
db.employees
  .find(
    { $text: { $search: "JavaScript React" } },
    { score: { $meta: "textScore" } },
  )
  .sort({ score: { $meta: "textScore" } });
```

### Explanation:

- `$meta: "textScore"` → exposes relevance score
- `.sort()` → ranks most relevant results first
- Documents matching both terms rank higher

## Advanced Features

### 1. Phrase Search

```javascript
db.employees.find({
  $text: { $search: '"Node.js MongoDB"' },
});
```

- Quotes enforce exact phrase matching

### 2. Exclusion Search

```javascript
db.employees.find({
  $text: { $search: "MongoDB -React" },
});
```

- `-React` excludes documents containing "React"

## Important Limitations

1. ❌ Only one text index per collection
2. ❌ Cannot use regex with `$text`
3. ❌ Cannot combine `$text` with another `$text` in same query
4. ❌ Score sorting required explicitly
5. ❌ Language limitations (default is English unless specified)

## Real-World Use Case

### Employee search system:

- Search resumes by skills
- Search job descriptions
- Search support tickets or logs
- Blog/article search engines

## Performance Optimization Tips

- Always create a **compound text index carefully** (only necessary fields)
- Combine with filters using `$and` where possible:

```javascript
db.employees.find({
  department: "Engineering",
  $text: { $search: "MongoDB" },
});
```

- Ensure index size is controlled (text indexes can grow large)

## Common Pitfalls

❌ Forgetting to create text index → query fails
❌ Expecting substring matching (it is word-based, not regex-based)
❌ Using multiple text indexes → not allowed

## Follow-up Interview Questions

### 1. What is the difference between `$text` search and Atlas Search?

- `$text`: basic built-in MongoDB search engine
- Atlas Search: Lucene-based, advanced ranking, fuzzy search, autocomplete

### 2. Can `$text` be used in aggregation pipelines?

Yes, using `$match` stage:

```javascript
db.employees.aggregate([
  { $match: { $text: { $search: "MongoDB" } } },
  { $project: { name: 1, score: { $meta: "textScore" } } },
]);
```
