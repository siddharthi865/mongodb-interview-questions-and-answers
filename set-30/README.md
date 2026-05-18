# Set 30

| S.No. | Question                                                                                                 |
| ----- | -------------------------------------------------------------------------------------------------------- |
| 1.    | [Design a digital wallet with ACID guarantees](#question-1-design-a-digital-wallet-with-acid-guarantees) |

## Question 1. Design a digital wallet with ACID guarantees

### 1. Core Idea (Interview Definition)

A **digital wallet system** is a financial system that allows users to store money, transfer funds, and track transactions. To ensure correctness, especially under concurrent transfers, we need **ACID guarantees (Atomicity, Consistency, Isolation, Durability)**.

In MongoDB (≥ 4.0+, recommended 6.x+), ACID is achieved using:

- **Multi-document transactions**
- **Replica set durability**
- **Write concern (`majority`)**

### 2. High-Level Data Model

We design around **ledger-based accounting** (preferred over just updating balances).

### Collections

#### `wallets`

```js
{
  _id: ObjectId(),
  userId: ObjectId(),
  balance: 1000,
  currency: "INR",
  updatedAt: ISODate()
}
```

#### `transactions`

```js
{
  _id: ObjectId(),
  fromWalletId: ObjectId(),
  toWalletId: ObjectId(),
  amount: 200,
  status: "SUCCESS", // PENDING | SUCCESS | FAILED
  createdAt: ISODate()
}
```

#### `ledger`

(Immutable source of truth — critical for audit & recovery)

```js
{
  _id: ObjectId(),
  walletId: ObjectId(),
  type: "DEBIT" | "CREDIT",
  amount: 200,
  referenceTransactionId: ObjectId(),
  createdAt: ISODate()
}
```

### 3. Core Transfer Operation (ACID Transaction)

### MongoDB Transaction Example

```js
const session = db.getMongo().startSession();

session.withTransaction(async () => {
  const wallets = session.getDatabase("interview_db").wallets;
  const transactions = session.getDatabase("interview_db").transactions;
  const ledger = session.getDatabase("interview_db").ledger;

  const fromId = ObjectId("...AliceId...");
  const toId = ObjectId("...BobId...");
  const amount = 200;

  // 1. Debit sender (with balance check)
  const fromWallet = await wallets.findOne({ _id: fromId }, { session });

  if (fromWallet.balance < amount) {
    throw new Error("Insufficient balance");
  }

  await wallets.updateOne(
    { _id: fromId },
    { $inc: { balance: -amount } },
    { session },
  );

  // 2. Credit receiver
  await wallets.updateOne(
    { _id: toId },
    { $inc: { balance: amount } },
    { session },
  );

  // 3. Create transaction record
  const tx = await transactions.insertOne(
    {
      fromWalletId: fromId,
      toWalletId: toId,
      amount,
      status: "SUCCESS",
      createdAt: new Date(),
    },
    { session },
  );

  // 4. Ledger entries (immutable audit log)
  await ledger.insertMany(
    [
      {
        walletId: fromId,
        type: "DEBIT",
        amount,
        referenceTransactionId: tx.insertedId,
        createdAt: new Date(),
      },
      {
        walletId: toId,
        type: "CREDIT",
        amount,
        referenceTransactionId: tx.insertedId,
        createdAt: new Date(),
      },
    ],
    { session },
  );
});
```

### 4. How ACID is Ensured

#### Atomicity

All operations (debit, credit, ledger writes) succeed or fail together via **transaction session**.

#### Consistency

- Balance never goes negative (checked before update)
- Ledger always matches wallet state

#### Isolation

MongoDB transactions use **snapshot isolation**, preventing dirty reads.

#### Durability

With `writeConcern: "majority"` in replica sets, committed transactions survive failures.

### 5. Performance & Design Considerations

#### Indexing

```js
db.wallets.createIndex({ userId: 1 });
db.transactions.createIndex({ fromWalletId: 1, createdAt: -1 });
db.ledger.createIndex({ walletId: 1, createdAt: -1 });
```

#### Avoid hotspots

- Do NOT store global counters
- Avoid single-wallet contention (use sharding if needed)

#### Idempotency

Use a unique `clientTransactionId` to prevent double spending in retries.

### 6. Common Pitfalls

- ❌ Updating balance without transactions → race conditions
- ❌ Not using ledger → impossible audits
- ❌ Missing writeConcern → unsafe durability assumptions
- ❌ Long-running transactions → performance degradation

### 7. Scaling Strategy

- **Sharding key:** `walletId`
- **Ledger is append-only → easy horizontal scaling**
- Use **read replicas** for transaction history queries

### Follow-up Interview Questions

#### 1. How do you prevent double-spending in distributed wallet systems?

Use:

- Unique transaction IDs
- Atomic balance checks inside transactions
- Optional locking or optimistic concurrency control

#### 2. What happens if a transaction fails midway?

MongoDB automatically rolls back all operations in the session ensuring no partial updates persist.
