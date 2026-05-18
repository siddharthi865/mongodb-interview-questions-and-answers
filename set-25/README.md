# Set 25

| S.No. | Question                                                                                                          |
| ----- | ----------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you restore partial backups in production?](#question-1-how-do-you-restore-partial-backups-in-production) |

## Question 1. How do you restore partial backups in production?

Restoring partial backups in MongoDB production environments means recovering **specific databases, collections, or even subsets of data** without restoring the entire cluster. This is common during accidental deletions, corrupted collections, or tenant-specific recovery.

In modern MongoDB deployments, partial restores are typically done using:

1. `mongorestore` with namespace filters
2. Point-in-Time Recovery (PITR) in MongoDB Atlas/Ops Manager
3. Temporary restore environments for selective recovery
4. Aggregation-based data extraction after restore

### 1. Restore Specific Collections Using `mongorestore`

Suppose only the `employees` collection was deleted.

#### Backup Command

```bash
mongodump --db interview_db --collection employees --out /backup/may15
```

#### Partial Restore

```bash
mongorestore \
  --db interview_db \
  --collection employees \
  /backup/may15/interview_db/employees.bson
```

#### Explanation

- `--db` → target database
- `--collection` → restore only one collection
- `.bson` file contains collection data

This avoids overwriting the entire database.

### 2. Restore Only Selected Documents

In production, restoring an entire collection may overwrite newer data.

A safer strategy is:

1. Restore backup into a temporary database
2. Copy only required documents

#### Step 1 — Restore to Temp DB

```bash
mongorestore \
  --nsFrom="interview_db.employees" \
  --nsTo="temp_restore.employees" \
  /backup/may15/interview_db/employees.bson
```

#### Step 2 — Copy Required Documents

```javascript
use temp_restore;

const employee = db.employees.findOne({ name: "Alice" });

use interview_db;

db.employees.insertOne(employee);
```

### 3. Point-in-Time Recovery (PITR)

For production systems, PITR is the preferred approach.

MongoDB Atlas and Ops Manager allow restoring data to an exact timestamp using oplogs.

Example scenario:

- Accidental deletion at 10:05 AM
- Restore cluster to 10:04:59 AM

This minimizes data loss.

Key requirement:

- Continuous oplog backups enabled
- Sufficient oplog retention window

MongoDB internally replays oplog operations after base snapshot restoration.

### 4. Restore Specific Namespaces

MongoDB supports namespace filtering.

#### Restore Only Engineering Collections

```bash
mongorestore \
  --nsInclude="interview_db.employees" \
  --nsInclude="interview_db.departments" \
  /backup/may15
```

Useful for microservice-oriented databases.

### Production Best Practices

#### A. Never Restore Directly into Live Collections First

Always restore into:

- temporary database
- isolated cluster
- staging environment

Then validate data before merging.

#### B. Use `--drop` Carefully

```bash
mongorestore --drop
```

This deletes target collections before restore.

Dangerous in production because it removes current data.

#### C. Preserve Indexes

`mongorestore` restores indexes automatically from metadata files.

You can verify:

```javascript
db.employees.getIndexes();
```

#### D. Handle Replica Sets Properly

Restore procedure for replica sets:

1. Restore on PRIMARY only
2. Let replication sync secondaries

Avoid restoring independently on each node.

### Example Production Recovery Workflow

Accidental deletion of one employee record:

1. Restore backup into temp DB
2. Verify document integrity
3. Reinsert missing document
4. Audit oplog entries
5. Monitor replication lag

This minimizes downtime and avoids full rollback.

### Performance & Operational Considerations

- Use compressed backups (`--gzip`)
- Use parallel restore:

```bash
mongorestore --numInsertionWorkersPerCollection 4
```

- Monitor disk I/O during restore
- Avoid peak traffic windows

For large sharded clusters:

- Restore shard-wise
- Preserve chunk metadata carefully

### Common Pitfalls

| Pitfall                          | Impact                         |
| -------------------------------- | ------------------------------ |
| Restoring directly to production | Overwrites live data           |
| Using `--drop` blindly           | Data loss                      |
| Ignoring oplog window            | PITR impossible                |
| Restoring without indexes        | Severe performance degradation |
| Partial shard restore            | Inconsistent cluster state     |

### Follow-up Interview Questions

#### 1. How does MongoDB support Point-in-Time Recovery?

MongoDB uses oplog-based recovery where snapshot backups are replayed with oplog entries up to a target timestamp.

#### 2. What is the safest way to restore deleted documents in production?

Restore into a temporary database first, validate data, then selectively copy required documents into production.
