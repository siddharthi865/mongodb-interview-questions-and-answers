# Set 10

## Question 1. How do you use client-side encryption in MongoDB?

**Client-Side Field Level Encryption (CSFLE)** in MongoDB allows you to **encrypt sensitive data before it leaves the application**, ensuring that even MongoDB servers (and cloud providers) cannot read the encrypted fields.

This is different from:

- **Transport encryption (TLS)** → protects data in transit
- **At-rest encryption (WiredTiger / Atlas encryption)** → protects stored data
- **CSFLE** → protects data _before it reaches MongoDB_

Only the application with the encryption keys can decrypt the data.

## How CSFLE Works (High-Level Steps)

1. **Define sensitive fields** (e.g., salary, SSN)
2. **Create a Key Management System (KMS)**
   (AWS KMS, Azure Key Vault, GCP KMS, or local master key for dev)
3. **Generate a Data Encryption Key (DEK)** stored in MongoDB
4. **Use Client Encryption library** to encrypt fields automatically
5. **Insert/query encrypted data using encrypted client**

## Example Setup (MongoDB Node.js Driver)

### 1. Create Key Vault Database

```javascript
use encryption;
db.createCollection("keyVault");
```

Create a **unique index on keyAltNames**:

```javascript
db.getCollection("keyVault").createIndex(
  { keyAltNames: 1 },
  { unique: true, partialFilterExpression: { keyAltNames: { $exists: true } } },
);
```

### 2. Create Data Encryption Key (DEK)

Using MongoDB Client Encryption:

```javascript
const clientEncryption = new ClientEncryption(kmsProviders, {
  keyVaultNamespace: "encryption.keyVault",
  client: keyVaultClient,
});
```

Generate DEK:

```javascript
const dataKey = await clientEncryption.createDataKey("local", {
  keyAltNames: ["employeeDataKey"],
});
```

### 3. Define Encrypted Fields Schema

```javascript
const encryptedFieldsMap = {
  "interview_db.employees": {
    fields: [
      {
        path: "salary",
        bsonType: "int",
        keyId: dataKey,
        algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Random",
      },
    ],
  },
};
```

### Explanation

- `path`: field to encrypt (`salary`)
- `keyId`: encryption key (DEK)
- `algorithm`:
  - **Deterministic** → supports equality queries
  - **Random** → stronger security, no querying

### 4. Create Encrypted MongoDB Client

```javascript
const encryptedClient = new MongoClient(uri, {
  autoEncryption: {
    keyVaultNamespace: "encryption.keyVault",
    kmsProviders,
    schemaMap: encryptedFieldsMap,
  },
});
```

### 5. Insert Encrypted Data

```javascript
const db = encryptedClient.db("interview_db");

await db.collection("employees").insertOne({
  name: "Alice",
  department: "Engineering",
  salary: 75000,
});
```

Even though you pass plain `75000`, MongoDB stores it encrypted like:

```json
"salary": BinData(...)
```

### 6. Query Encrypted Data

```javascript
const result = await db.collection("employees").findOne({
  name: "Alice",
});
```

- Decryption happens **automatically on client side**
- Server never sees plaintext salary

## Key Features

### Strong Security

- Data is encrypted **before network transmission**
- MongoDB server cannot decrypt fields

### Query Support (Limited)

- Deterministic encryption → supports equality queries:

```javascript
{
  salary: 75000;
}
```

- Random encryption → not queryable

## Performance Considerations

- Encryption/decryption happens in **client application**
- Slight CPU overhead on application side
- Indexing is limited on encrypted fields
- Best used only for **highly sensitive fields**

## Common Pitfalls

- Losing access to KMS = **data permanently inaccessible**
- Cannot use normal indexes on encrypted fields
- Schema design must be planned carefully upfront
- Debugging is harder since data is opaque in DB

## Real-World Use Cases

- Banking systems (account balances)
- Healthcare (patient records)
- HR systems (salary, personal info)
- Government identity systems

## Follow-up Interview Questions

### 1. What is the difference between CSFLE and MongoDB Atlas Encryption at Rest?

- CSFLE encrypts **before data leaves application**
- At-rest encryption protects data **only on disk**

### 2. Can you run aggregation on encrypted fields?

- ❌ No, unless using limited deterministic equality operations
- Aggregation pipelines cannot decrypt server-side
