# Set 9

| S.No. | Question                                                                                                        |
| ----- | --------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you perform atomic operations in MongoDB?](#question-1-how-do-you-perform-atomic-operations-in-mongodb) |

## Question 1. How do you perform atomic operations in MongoDB?

## Atomic Operations in MongoDB (Interview Answer)

### Core Concept

In MongoDB, **atomic operations** ensure that a single write operation on a document is **fully completed or not applied at all**. MongoDB guarantees **atomicity at the single-document level**, meaning updates to one document are always atomic—even if multiple fields are modified.

For multi-document atomicity, MongoDB provides **transactions**.

## 1. Atomicity at Document Level (Default Behavior)

MongoDB guarantees that operations like `insertOne()`, `updateOne()`, `findOneAndUpdate()` on a single document are atomic.

### Example

```javascript
db.employees.updateOne(
  { name: "Alice" },
  {
    $set: { salary: 80000, department: "Engineering" },
    $inc: { salary: 5000 },
  },
);
```

### Explanation

- `{ name: "Alice" }` → Filter condition
- `$set` → Updates multiple fields atomically
- `$inc` → Increments salary safely
- Entire update happens as **one atomic operation on the document**

Even if the server crashes mid-operation, the document will never be partially updated.

## 2. Why Atomicity Matters

Atomic operations help in:

- Preventing **race conditions**
- Ensuring **data consistency**
- Avoiding partial updates
- Supporting concurrent writes safely

### Real-world use case

Updating a bank balance:

```javascript
db.accounts.updateOne({ accountId: 101 }, { $inc: { balance: -500 } });
```

Debit operation is atomic, so balance cannot become inconsistent.

## 3. Multi-Document Atomicity (Transactions)

For operations involving **multiple documents or collections**, MongoDB uses **ACID transactions** (available in replica sets and sharded clusters).

### Example Transaction

```javascript
const session = db.getMongo().startSession();

session.startTransaction();

try {
  const employees = session.getDatabase("interview_db").employees;
  const logs = session.getDatabase("interview_db").logs;

  employees.updateOne({ name: "Bob" }, { $inc: { salary: 5000 } });

  logs.insertOne({
    message: "Salary updated for Bob",
    date: new Date(),
  });

  session.commitTransaction();
} catch (error) {
  session.abortTransaction();
}
```

### Explanation

- `startSession()` → Begins session context
- `startTransaction()` → Begins ACID transaction
- Multiple operations execute together
- `commitTransaction()` → Saves all changes
- `abortTransaction()` → Rolls back everything if failure occurs

Ensures **all-or-nothing behavior across collections**

## 4. Key Limitations & Best Practices

### Limitations

- Single-document atomicity only (without transactions)
- Transactions add **performance overhead**
- Must be used only when necessary

### Best Practices

- Prefer **embedded document design** to avoid transactions
- Use atomic operators: `$set`, `$inc`, `$push`, `$pull`
- Use transactions only for cross-document consistency needs

## 🔹 5. Performance Optimization Insight

From MongoDB docs: transactions should be:

- Short-lived
- Small in scope
- Avoid large batch updates inside transactions

## Follow-up Interview Questions

### 1. When should you avoid using transactions in MongoDB?

When data can be modeled as a single document or when performance is critical and cross-document consistency is not required.

### 2. How does MongoDB ensure atomicity at the document level internally?

It uses an internal write lock per document update operation, ensuring no partial updates occur during execution.
