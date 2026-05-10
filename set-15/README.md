# Set 15

| S.No. | Question                                                                                                                    |
| ----- | --------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you perform point-in-time recovery for MongoDB?](#question-1-how-do-you-perform-point-in-time-recovery-for-mongodb) |

## Question 1. How do you perform point-in-time recovery for MongoDB?

**Point-in-Time Recovery (PITR)** in MongoDB is a backup and restore strategy that allows you to restore a database to a **specific moment in time**, rather than just the last full backup. This is critical for recovering from accidental deletes, bad deployments, or data corruption.

In MongoDB, PITR is primarily supported via **MongoDB Atlas Continuous Backup** or manually using **oplog-based backups in replica sets**.

MongoDB achieves PITR using:

- **Base Snapshot** → A full backup at a point in time
- **Oplog (Operations Log)** → A continuous log of all write operations
- **Replay mechanism** → Reapply oplog entries up to a target timestamp

So the restore flow is:

> Snapshot + Replay oplog up to timestamp = Point-in-time state

## 1. PITR in MongoDB Atlas (Recommended Approach)

In **MongoDB Atlas (official managed service)**, PITR is built-in.

### Steps

1. Enable **Continuous Cloud Backup**
2. Atlas automatically:
   - Takes snapshots periodically
   - Continuously stores oplog

3. Select:
   - Cluster
   - Snapshot
   - Target timestamp

4. Restore to a new cluster or overwrite existing one

### Example (Atlas UI / API concept)

You choose:

- Snapshot: `2026-05-14T10:00:00Z`
- Target time: `2026-05-14T10:37:12Z`

Atlas:

- Restores snapshot
- Applies oplog entries up to 10:37:12

## 2. PITR in Self-Managed MongoDB (Replica Set)

For self-managed clusters, PITR is implemented using:

- `mongodump`
- `mongorestore`
- **oplog replay**

## Step 1: Take a Full Backup with Oplog

```bash
mongodump --uri="mongodb://localhost:27017" \
  --out=/backup/full \
  --oplog
```

### Explanation

- `mongodump` → Creates backup
- `--oplog` → Captures operations during backup window (critical for PITR)

## Step 2: Restore Snapshot

```bash
mongorestore --drop --dir=/backup/full
```

### Explanation

- `--drop` → Clears existing data before restore
- Restores base snapshot first

## Step 3: Apply Oplog to a Specific Time

```bash
mongorestore \
  --dir=/backup/full \
  --oplogReplay \
  --oplogLimit "2026-05-14T10:37:12"
```

### Explanation

- `--oplogReplay` → Re-applies operations
- `--oplogLimit` → Stops at specific timestamp → THIS enables PITR

## 3. How PITR Works Internally (Important Interview Insight)

MongoDB replica sets maintain:

### Oplog Collection

- Stored in `local.oplog.rs`
- A capped collection
- Records every write operation

Example oplog entry:

```javascript
{
  ts: Timestamp(1715670000, 1),
  op: "i",
  ns: "interview_db.employees",
  o: { name: "David", salary: 90000 }
}
```

### Fields

- `ts` → timestamp of operation
- `op` → operation type (`i`, `u`, `d`)
- `ns` → namespace (db.collection)
- `o` → operation payload

## Use Cases of PITR

- Accidental `deleteMany()` without filter
- Wrong migration script deployed
- Corrupted data ingestion pipeline
- Ransomware or malicious write operations
- Recovering to exact pre-bug state in production

## Performance & Best Practices

### 1. Ensure sufficient oplog size

```javascript
rs.printReplicationInfo();
```

Large oplog window = better recovery granularity.

### 2. Use Atlas Continuous Backup for production

- Eliminates manual oplog management
- Reduces recovery time (RTO)

### 3. Store backups in separate region/account

Prevents single-point failure.

### 4. Test restores regularly

Many teams fail PITR because restore is never tested.

# Common Pitfalls

- Oplog too small → cannot reach desired timestamp
- Not enabling `--oplog` during backup
- Time mismatch (UTC vs local time confusion)
- Missing index rebuild expectations after restore

## MongoDB Docs Reference

- [https://www.mongodb.com/docs/manual/core/replica-set-oplog/](https://www.mongodb.com/docs/manual/core/replica-set-oplog/)
- [https://www.mongodb.com/docs/atlas/backup-restore-cluster/](https://www.mongodb.com/docs/atlas/backup-restore-cluster/)

## Follow-up Interview Questions

### 1. What happens if the oplog window is smaller than the recovery range?

You can only restore up to the earliest oplog entry available; PITR beyond that point is impossible.

### 2. How is PITR different in sharded clusters?

Each shard has its own oplog; Atlas coordinates snapshot + oplog replay across shards to maintain consistency.
