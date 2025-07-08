Welcome to the underworld of PostgreSQL internals. This is where the ghosts of queries past and present hang out. But `pg_stat_activity` is just the tip of the observability iceberg. Here’s a curated list of **PostgreSQL system views and stats tables** you should be checking periodically if you want to be the DB whisperer.

---

## 🧠 Core Monitoring Views Like `pg_stat_activity`

### 1. **`pg_stat_activity`**

* **Why**: Shows who is doing what — running queries, waiting, sleeping, blocking others.
* **Check for**: Long-running queries, idle in transaction states, connection bloat.

---

## 📈 Performance and IO Insight Tables

### 2. **`pg_stat_user_tables`**

* **Why**: Track table-level stats: seq scans, idx scans, updates, inserts, dead tuples.
* **Check for**: Tables getting a lot of writes or no index usage (bad for performance).

### 3. **`pg_stat_user_indexes`**

* **Why**: Shows how often indexes are used.
* **Check for**: Unused or bloated indexes hogging disk space.

### 4. **`pg_statio_user_tables`**

* **Why**: Shows **IO-level** stats (heap, toast, index blocks read/hit).
* **Check for**: Buffer cache misses (too many reads = cache not helping = slow).

---

## 🔄 Autovacuum & Bloat Watch

### 5. **`pg_stat_all_tables`**

* **Why**: Same as `pg_stat_user_tables` but includes system tables.
* **Check for**: Autovacuum status and dead tuples (VACUUM needed).

### 6. **`pg_stat_bgwriter`**

* **Why**: Insight into the background writer's workload.
* **Check for**: Excessive checkpoints or buffer writes (tune your `shared_buffers`, `checkpoint_timeout`, etc.).

---

## 🔒 Locks, Waits, and Deadlocks

### 7. **`pg_locks`**

* **Why**: Real-time view of locks held or waiting.
* **Check for**: Blocking sessions, lock contention.

### 8. **`pg_stat_activity` + `pg_locks` JOIN**

* **Why**: Understand **who is blocking whom** and **why**.
* **Check for**: Deadlocks, especially in high-write systems.

---

## 🧹 Bloat & Maintenance Candidates

### 9. **`pg_stat_all_indexes`**

* **Why**: Index-level stats including scans and size.
* **Check for**: Unused but large indexes. Combine with `pg_statio_user_indexes`.

### 10. **`pg_tables`, `pg_indexes`, `pg_class` + `pg_size_pretty(pg_total_relation_size(...))`**

* **Why**: See actual table/index size (not just row count).
* **Check for**: Tables that are huge but rarely used or have poor write/read ratios.

---

## 🧪 Others (Power Tools)

### 11. **`pg_prepared_xacts`**

* **Why**: Two-phase commits hanging around?
* **Check for**: Forgotten prepared transactions that need cleanup.

### 12. **`pg_stat_replication`**

* **Why**: Replication lag, streaming status.
* **Check for**: Replica lag if you're running HA/Postgres streaming replication.

### 13. **`pg_stat_database`**

* **Why**: Summary stats per DB: commits, rollbacks, temp files, deadlocks.
* **Check for**: Dirty DBs, high temp usage (maybe missing work\_mem tuning).

---

## 👀 How to Use These in Practice

* **Weekly**: Check `pg_stat_user_tables`, `pg_stat_user_indexes`, `pg_stat_activity`.
* **Daily** (for prod): Check for blocking queries in `pg_stat_activity` + `pg_locks`.
* **Monthly**: Run bloat checks (via extensions or manual queries).
* **When Things Go Sideways™**: Combine all of the above to triage the situation fast.

---


