# Set 3

| S.No. | Question                                                                                     |
| ----- | -------------------------------------------------------------------------------------------- |
| 1.    | [How do you use $project in aggregation?](#question-1-how-do-you-use-project-in-aggregation) |

## Question 1. How do you use $project in aggregation?

`$project` is an aggregation stage in MongoDB used to **shape the output documents**.
It allows you to:

- Include specific fields
- Exclude fields
- Rename fields
- Create computed fields
- Transform existing data

It is similar to the `SELECT` clause in SQL.

### Basic Syntax

```javascript
{
  $project: {
    field1: 1,
    field2: 1,
    fieldToExclude: 0,
    newField: <expression>
  }
}
```

- `1` → include field
- `0` → exclude field
- Expressions → create calculated fields

### Example Using Sample Dataset

```javascript
use interview_db;

db.employees.aggregate([
  {
    $project: {
      name: 1,
      department: 1,
      salary: 1
    }
  }
]);
```

### Explanation

### Stage 1 — `$project`

```javascript
{
  $project: {
    name: 1,
    department: 1,
    salary: 1
  }
}
```

#### What it does

- Includes only:
  - `name`
  - `department`
  - `salary`

- Excludes all other fields automatically.

#### Output

```javascript
{
  _id: ObjectId("..."),
  name: "Alice",
  department: "Engineering",
  salary: 75000
}
```

### Excluding `_id`

MongoDB includes `_id` by default.

To remove it:

```javascript
db.employees.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      department: 1,
    },
  },
]);
```

### Output

```javascript
{
  name: "Alice",
  department: "Engineering"
}
```

### Creating Computed Fields

You can generate new fields dynamically.

```javascript
db.employees.aggregate([
  {
    $project: {
      name: 1,
      annualBonus: {
        $multiply: ["$salary", 0.1],
      },
    },
  },
]);
```

### Explanation

```javascript
annualBonus: {
  $multiply: ["$salary", 0.1];
}
```

- `$salary` references the document field
- `$multiply` calculates 10% bonus

### Output

```javascript
{
  name: "Alice",
  annualBonus: 7500
}
```

### Renaming Fields

```javascript
db.employees.aggregate([
  {
    $project: {
      employeeName: "$name",
      dept: "$department",
      _id: 0,
    },
  },
]);
```

### Output

```javascript
{
  employeeName: "Alice",
  dept: "Engineering"
}
```

### Using Expressions in `$project`

MongoDB supports many operators:

- `$concat`
- `$toUpper`
- `$substr`
- `$ifNull`
- `$cond`
- `$dateToString`

Example:

```javascript
db.employees.aggregate([
  {
    $project: {
      name: 1,
      upperDept: {
        $toUpper: "$department",
      },
    },
  },
]);
```

### Performance Considerations

#### 1. Reduce Data Early

Place `$project` early in the pipeline to reduce unnecessary fields.

```javascript
[{ $project: { largeField: 0 } }];
```

This reduces memory usage and improves pipeline efficiency.

#### 2. Avoid Mixing Include and Exclude

MongoDB does not allow mixing inclusion and exclusion in the same `$project` stage except for `_id`.

❌ Invalid:

```javascript
{
  $project: {
    name: 1,
    salary: 0
  }
}
```

#### 3. Use `$unset` for Simple Field Removal

If only removing fields, `$unset` can be cleaner.

```javascript
{
  $unset: ["managerId"];
}
```

#### Real-World Use Cases

- API response formatting
- Hiding sensitive fields
- Generating reports
- Creating calculated values
- Reshaping documents before `$group`

### Common Interview Follow-Up Questions

#### 1. What is the difference between `$project` and `$addFields`?

- `$project` reshapes documents and can remove fields.
- `$addFields` only adds/modifies fields while keeping existing fields intact.

#### 2. Can `$project` improve performance?

Yes. Reducing document size early in the pipeline decreases memory usage and improves aggregation efficiency.
