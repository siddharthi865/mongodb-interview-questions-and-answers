# Set 18

| S.No. | Question                                                                                                                                   |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you perform a self-referencing join using $graphLookup?](#question-1-how-do-you-perform-a-self-referencing-join-using-graphlookup) |

## Question 1. How do you perform a self-referencing join using $graphLookup?

A self-referencing join in MongoDB is commonly used to model **hierarchical or recursive relationships** such as:

- Employee → Manager hierarchy
- Categories → Parent categories
- Organization trees
- Comments → Replies

MongoDB provides the `$graphLookup` aggregation stage to recursively traverse documents within the **same collection**.

### Using `$graphLookup` for Self-Referencing Joins

Suppose we have an `employees` collection where each employee stores their manager’s `_id`.

#### Sample Data

```javascript
use interview_db;

const aliceId = ObjectId();
const bobId = ObjectId();
const charlieId = ObjectId();

db.employees.insertMany([
  {
    _id: aliceId,
    name: "Alice",
    department: "Engineering",
    managerId: null
  },
  {
    _id: bobId,
    name: "Bob",
    department: "Engineering",
    managerId: aliceId
  },
  {
    _id: charlieId,
    name: "Charlie",
    department: "Engineering",
    managerId: bobId
  }
]);
```

Hierarchy:

```text
Alice
 └── Bob
      └── Charlie
```

### Example: Find Complete Reporting Chain

```javascript
db.employees.aggregate([
  {
    $match: {
      name: "Charlie",
    },
  },
  {
    $graphLookup: {
      from: "employees",
      startWith: "$managerId",
      connectFromField: "managerId",
      connectToField: "_id",
      as: "managementChain",
      depthField: "level",
    },
  },
]);
```

### Explanation Line-by-Line

#### 1. `$match`

```javascript
{
  $match: {
    name: "Charlie";
  }
}
```

Starts aggregation from Charlie’s document.

#### 2. `$graphLookup`

```javascript
{
  $graphLookup: {
```

Performs recursive traversal.

##### `from`

```javascript
from: "employees";
```

Target collection for recursive lookup.

Since this is self-referencing, it points to the same collection.

##### `startWith`

```javascript
startWith: "$managerId";
```

Initial value to begin traversal.

For Charlie, this starts with Bob’s `_id`.

##### `connectFromField`

```javascript
connectFromField: "managerId";
```

Field used recursively from matched documents.

MongoDB keeps following `managerId`.

##### `connectToField`

```javascript
connectToField: "_id";
```

Matches `managerId` to employee `_id`.

##### `as`

```javascript
as: "managementChain";
```

Stores recursive results in an array.

##### `depthField`

```javascript
depthField: "level";
```

Adds recursion depth metadata:

```text
Bob     → level 0
Alice   → level 1
```

### Example Output

```javascript
{
  _id: ObjectId("..."),
  name: "Charlie",
  managerId: ObjectId("..."),
  managementChain: [
    {
      name: "Bob",
      level: 0
    },
    {
      name: "Alice",
      level: 1
    }
  ]
}
```

### Important `$graphLookup` Features

| Feature                      | Purpose                          |
| ---------------------------- | -------------------------------- |
| Recursive traversal          | Walks hierarchical relationships |
| Same or different collection | Works for both                   |
| `maxDepth`                   | Limits recursion                 |
| `depthField`                 | Tracks hierarchy depth           |
| `restrictSearchWithMatch`    | Filters recursive results        |

### Performance Optimization

#### 1. Index Critical Fields

Always index recursive join fields:

```javascript
db.employees.createIndex({ managerId: 1 });
```

This is essential because `$graphLookup` repeatedly searches using `managerId`.

#### 2. Use `maxDepth`

Prevents infinite or expensive traversals.

```javascript
maxDepth: 3;
```

Useful for deep hierarchies.

#### 3. Avoid Cyclic References

Bad data can create loops:

```text
Alice → Bob → Charlie → Alice
```

MongoDB internally prevents infinite recursion, but cycles still hurt performance.

### Advanced Example with `maxDepth`

```javascript
db.employees.aggregate([
  {
    $graphLookup: {
      from: "employees",
      startWith: "$managerId",
      connectFromField: "managerId",
      connectToField: "_id",
      as: "hierarchy",
      maxDepth: 2,
      depthField: "depth",
    },
  },
]);
```

### Real-World Use Cases

- Org chart systems
- Folder structures
- Product category trees
- Social network connections
- Dependency graphs

### Common Interview Follow-Up Questions

#### 1. What is the difference between `$lookup` and `$graphLookup`?

- `$lookup` performs a one-level join.
- `$graphLookup` performs recursive joins across multiple levels.

#### 2. Can `$graphLookup` work across collections?

Yes. It can recursively traverse documents in another collection using matching fields.

For example:

- Users collection
- Roles collection
- Category collections

MongoDB Docs:
[MongoDB $graphLookup Documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation/graphLookup)
