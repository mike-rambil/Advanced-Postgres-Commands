# Postgres Backups: Dump, Physical, WAL/PITR, Tooling, Pitfalls

A general reference for backing up PostgreSQL: the vocabulary, the methods Postgres gives you, the tools people actually run in production, and the mistakes that turn a "backup" into a false sense of security.

> [!TIP]
> For a quick side-by-side of `pg_dump` files vs WAL-G, see [diff_between_WAL-G__vs__dumpFile.md](./diff_between_WAL-G__vs__dumpFile.md).

### Table of Contents

1. [Key Metrics: RPO and RTO](#key-metrics-rpo-and-rto)
2. [Classic Backup Types](#classic-backup-types)
3. [Deduplicating Snapshot Tools](#deduplicating-snapshot-tools)
4. [WAL (Write-Ahead Log)](#wal-write-ahead-log)
5. [Backup Methods](#backup-methods)
6. [Industry Standard](#industry-standard)
7. [Common Pitfalls](#common-pitfalls)
8. [Choosing a Strategy by Scale](#choosing-a-strategy-by-scale)

---

## Key Metrics: RPO and RTO

* **RPO (Recovery Point Objective)**: how much data loss is acceptable. A nightly backup means you can lose up to ~24h of writes.
* **RTO (Recovery Time Objective)**: how long a restore is allowed to take, from "it's broken" to "it's serving traffic again".

Every decision below is a trade-off between these two numbers and how much complexity you're willing to run.

---

## Classic Backup Types

| Type         | Stores                                       | Restore needs                                  |
| ------------ | -------------------------------------------- | ---------------------------------------------- |
| Full         | Everything, every time                       | That one backup                                |
| Incremental  | Changes since the last backup of *any* kind  | Full + every incremental after it (a chain)    |
| Differential | Changes since the last **full**              | Full + the latest differential                 |

* **Watch out**: incremental chains are fragile. One corrupt link breaks every restore point after it.

---

## Deduplicating Snapshot Tools

Tools like **restic**, **Borg** and **Kopia** don't fit neatly into the table above.

* They use **content-defined chunking + deduplication** (content-addressable storage), often called "forever-incremental" or "synthetic full".
* Files are split into variable-size chunks, each named by its hash. Chunks already in the repository aren't uploaded again.
* Each snapshot is a list of chunk IDs that together form a complete picture.
  * **Uploads** behave like incrementals (only new chunks are sent).
  * **Restores** behave like fulls (no chain to replay).
* **Consequences**:
  * Never delete repository files by age (e.g. object-storage lifecycle rules). Newer snapshots still reference old chunks.
  * Use the tool's own retention instead:

    ```bash
    restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
    ```

* **Note**: these tools back up *files*. Point them at a dump or a proper base backup, never at a live data directory (see [Common Pitfalls](#common-pitfalls)).

---

## WAL (Write-Ahead Log)

* Postgres writes every change to the WAL (`pg_wal/`, 16 MB segments by default) **before** touching the data files.
* **Uses**:
  1. **Crash recovery**: replay the WAL since the last checkpoint after a crash.
  2. **Replication**: replicas stream and replay the WAL.
  3. **PITR (Point-In-Time Recovery)**: archive every segment, then replay up to any moment.
* **How it differs from dedup tools**:
  * Dedup works at the **storage** level: it deduplicates bytes.
  * WAL works at the **database** level: it's a log of every transaction.
  * So WAL can restore to an exact second, e.g. just before a bad `DELETE`.

---

## Backup Methods

### A. Logical: `pg_dump` / `pg_dumpall`

```bash
pg_dump -Fc -d mydb -f mydb.dump          # custom format, compressed
pg_dump -Fd -j4 -d mydb -f mydb.dir       # directory format, parallel dump
pg_dumpall --globals-only > globals.sql   # roles and tablespaces (pg_dump skips them)
pg_restore -d mydb -j4 mydb.dump          # parallel restore
```

* **Pros**:
  * Portable across machines and to newer major versions.
  * Can restore a single table or schema.
  * Consistent: the whole database is dumped from one MVCC snapshot.
* **Cons**:
  * RPO equals the dump interval.
  * Slow for large databases.
  * Restore rebuilds every index.
* **Good fit**: small and medium databases, and major-version upgrades.

### B. Physical: `pg_basebackup`

```bash
pg_basebackup -D /backups/base -Ft -z -Xstream -P
```

* A byte-level copy of the whole cluster while it runs.
* Fast restore, and the foundation for PITR.
* **Limits**: whole cluster only, same major version, same architecture.

### C. Physical + Continuous WAL Archiving = PITR

* A periodic base backup, plus every WAL segment shipped off the server via `archive_command` (or `archive_library`, PG 15+).

  ```ini
  # postgresql.conf
  wal_level = replica
  archive_mode = on
  archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
  ```

* To restore: lay down the base backup, set `restore_command` and a target, then create `recovery.signal` (PG 12+).

  ```ini
  restore_command = 'cp /archive/%f %p'
  recovery_target_time = '2025-01-15 14:29:00+00'
  ```

* RPO of seconds to minutes.
* **Cost**: more moving parts, and the archive must have no gaps. One missing segment stops replay at that point.

### D. PostgreSQL 17 Native Incremental Backups

```bash
# postgresql.conf: summarize_wal = on
pg_basebackup -D /backups/full -c fast
pg_basebackup -D /backups/incr1 --incremental=/backups/full/backup_manifest
pg_combinebackup /backups/full /backups/incr1 -o /restore/data
```

* `pg_combinebackup` reconstructs a full data directory from the chain.
* New, and less battle-tested than dedicated tools.

### E. Filesystem / Volume Snapshots

* LVM, ZFS, or cloud disk snapshots.
* Safe **only** if atomic and covering the data directory and `pg_wal` together (one consistent snapshot across all volumes).
* Otherwise wrap the snapshot in `pg_backup_start()` / `pg_backup_stop()` (named `pg_start_backup` / `pg_stop_backup` before PG 15).

---

## Industry Standard

Physical base backups + continuous WAL archiving to object storage, managed by a dedicated tool:

| Tool       | Notes                                                                                    |
| ---------- | ---------------------------------------------------------------------------------------- |
| pgBackRest | Full/diff/incr, parallel, encryption, S3-compatible storage, verification, retention     |
| WAL-G      | Lightweight, cloud-native, popular in containers and Kubernetes                          |
| Barman     | From EDB, common in traditional setups                                                   |

* **Managed services** (RDS, Cloud SQL, Neon, Supabase) do this internally. That's what "PITR with N-day retention" means.
* **Plus**:
  * Periodic logical dumps for portability and single-table restores.
  * The **3-2-1 rule**: 3 copies, 2 media types, 1 off-site.
  * Scheduled, timed restore drills.

---

## Common Pitfalls

1. **Never testing restores.** An untested backup is a hope. Schedule drills and time them; that measures your real RTO.
2. **Copying a running data directory** with `rsync` / `tar` / `restic`. Files change mid-copy, so the copy is corrupt. Back up a dump or a proper base backup instead. SQLite has the same problem: use `sqlite3 .backup` or the backup API.
3. **Treating replicas as backups.** `DROP TABLE` replicates instantly. Replicas give HA, not backups.
4. **Forgetting globals.** `pg_dump` omits roles and passwords; run `pg_dumpall --globals-only` too.
5. **A silently failing WAL archive.** Postgres keeps unarchived WAL, `pg_wal` fills the disk, and the database stops. Monitor `pg_stat_archiver`. `archive_command` must return non-zero on failure and must not overwrite existing files.
6. **Backups in the same blast radius** as the data: same disk, same server, same cloud account.
7. **Credentials on the server that can delete backups.** A compromised host or ransomware can wipe them. Use object lock / bucket lock, append-only credentials, or run pruning from a separate host.
8. **Losing the encryption key or password**, which makes encrypted backups unrecoverable. Store it somewhere other than the server.
9. **Retention shorter than the time it takes to notice corruption.** Keep weekly and monthly backups, not only daily.
10. **No monitoring.** Use dead-man's-switch alerts that fire when *no* success is reported.
11. **`pg_dump` client older than the server.** Use the same or a newer client version.
12. **Measuring RTO for the first time during a real incident.**

---

## Choosing a Strategy by Scale

| Situation                            | Strategy                                                                                                                              |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| Small DB, ~24h RPO acceptable        | Scheduled `pg_dump` + `pg_dumpall --globals-only`, shipped off-site (e.g. restic to object storage) with retention, monitoring and tested restores |
| Need minutes of RPO, or a large DB   | pgBackRest or WAL-G: base backups + WAL archiving (PITR)                                                                              |
| Large DB, need fast restore          | Physical backups / PITR plus a standby replica for HA, still with off-site backups                                                    |
| SQLite apps                          | `sqlite3 .backup` snapshots; Litestream for continuous streaming (the WAL-archiving equivalent)                                       |
