# Set 22

| S.No. | Question                                                                                                    |
| ----- | ----------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you use $first and $last in aggregation?](#question-1-how-do-you-use-first-and-last-in-aggregation) |

## Question 1. How do you use $first and $last in aggregation?

In MongoDB aggregation, `$first` and `$last` are accumulator operators used mainly inside the `$group` stage to return the **first** or **last** document value from a grouped set.

They are commonly used for:

- Finding earliest/latest records
- Getting first/last transaction
- Retrieving min/max ordered values after sorting
- Time-series analytics

### Key Concept

`$first` and `$last` depend on the **document order entering the `$group` stage**.

So in real-world usage, you almost always use `$sort` before `$group`.

### Example Dataset

```javascript
use interview_db;

db.employees.insertMany([
  {
    name: "Alice",
    department: "Engineering",
    salary: 75000,
    hireDate: ISODate("2020-01-15")
  },
  {
    name: "Bob",
    department: "Engineering",
    salary: 80000,
    hireDate: ISODate("2019-03-10")
  },
  {
    name: "Charlie",
    department: "HR",
    salary: 60000,
    hireDate: ISODate("2021-07-22")
  }
]);
```

### Example: Find First and Last Employee Hired Per Department

```javascript
db.employees.aggregate([
  {
    $sort: {
      hireDate: 1,
    },
  },
  {
    $group: {
      _id: "$department",

      firstEmployee: {
        $first: "$name",
      },

      lastEmployee: {
        $last: "$name",
      },

      firstHireDate: {
        $first: "$hireDate",
      },

      lastHireDate: {
        $last: "$hireDate",
      },
    },
  },
]);
```

### Step-by-Step Explanation

#### 1. `$sort`

```javascript
{
  $sort: {
    hireDate: 1;
  }
}
```

This sorts employees by ascending hire date.

Order becomes:

1. Bob → 2019
2. Alice → 2020
3. Charlie → 2021

This step is critical because `$first` and `$last` rely on pipeline order.

#### 2. `$group`

```javascript
{
  $group: {
    _id: "$department";
  }
}
```

Groups documents by department.

#### 3. `$first`

```javascript
firstEmployee: {
  $first: "$name";
}
```

Returns the first employee encountered in each department group.

For Engineering:

- Bob is first after sorting.

#### 4. `$last`

```javascript
lastEmployee: {
  $last: "$name";
}
```

Returns the last employee encountered.

For Engineering:

- Alice becomes last.

### Expected Output

```javascript
[
  {
    _id: "Engineering",
    firstEmployee: "Bob",
    lastEmployee: "Alice",
    firstHireDate: ISODate("2019-03-10"),
    lastHireDate: ISODate("2020-01-15"),
  },
  {
    _id: "HR",
    firstEmployee: "Charlie",
    lastEmployee: "Charlie",
    firstHireDate: ISODate("2021-07-22"),
    lastHireDate: ISODate("2021-07-22"),
  },
];
```

### Performance Considerations

#### 1. Always Sort Explicitly

Without `$sort`, results are unpredictable because MongoDB does not guarantee document order.

Bad practice:

```javascript
db.employees.aggregate([
  {
    $group: {
      _id: "$department",
      firstEmployee: { $first: "$name" },
    },
  },
]);
```

#### 2. Use Indexes for Sorting

For better performance:

```javascript
db.employees.createIndex({
  hireDate: 1,
  department: 1,
});
```

This helps MongoDB optimize the sort stage.

### Advanced Usage

Starting in modern MongoDB versions, `$first` and `$last` are also supported in:

- `$setWindowFields`
- Window functions
- Time-series analytics

Example use cases:

- First purchase by customer
- Latest sensor reading
- Running analytics

### Common Interview Pitfall

Many candidates forget:

> `$first` and `$last` do NOT mean minimum or maximum values automatically.

They return values based on incoming document order.

### Follow-up Interview Questions

#### 1. What is the difference between `$first` and `$min`?

- `$first` returns the first document value based on order.
- `$min` returns the minimum value mathematically.

#### 2. Can `$first` be used without `$group`?

Yes. In newer MongoDB versions, it can also be used in:

- `$setWindowFields`
- Window operators
- Some array expressions in projection stages
