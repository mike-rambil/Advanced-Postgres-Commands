

---

## 🔒 1. **Rethinking `SELECT FOR UPDATE` in PostgreSQL**

* **Problem**: `SELECT FOR UPDATE` can cause unnecessary locking.
* **Impact**: It **blocks inserts** on child tables with foreign keys.
* **Fix**: Use `SELECT FOR NO KEY UPDATE` instead (lighter lock unless you’re updating a primary/foreign key).
* **Reminder**: ORMs may default to heavier locks. Check your ORM behavior.

---

## 🔍 2. **Full-Text Search: Postgres vs Elasticsearch vs PG Search**

* **PostgreSQL FTS**: Low-level, verbose, performance degrades at scale.
* **Elasticsearch**: High performance, BM25 ranking, fuzzy matching — but adds complexity (separate DB).
* **PG Search** (extension): Embedded in Postgres, BM25 indexes, outperforms FTS 20x–1000x in benchmarks.

---

## 📥 3. **Primary Keys During Bulk Loads**

* **Claim**: Keeping **primary keys** during bulk inserts may be faster.
* **Test Results**:

  * Dropping PK → 10m insert + 20m rebuild = 30m total.
  * Keeping PK → 20m insert = ✅ Faster overall.
* **Lesson**: It’s not always worth removing PKs unless other indexes/constraints are also dropped.

---

## 🦆 4. **PostgreSQL + DuckDB = Analytical Powerhouse**

* **DuckDB Integration Options**:

  1. **DuckDB extension that connects to Postgres** (via connection string).
  2. **Pushdown queries** from DuckDB.
  3. **`pgduckdb` extension inside Postgres** — lets you query columnar formats like Parquet, Iceberg.
* **Pro Tips**:

  * Don’t run DuckDB analytical loads on the **primary** DB. Use a **replica**.
  * Major speed-ups seen in analytical queries (up to 1500x faster).

---

## 🔁 5. **Do You Really Need Active-Active (Multi-Master) Replication?**

* **When it's useful**:

  * Multi-region **write availability**
  * **Disaster recovery**
  * **Global performance** (lower latency)
  * **Legacy distributed workloads**
* **Caveat**: Eventual consistency = complex conflict handling. Few real-world use cases seen.

---

## 📊 6. **Postgres 18: Optimizer Stats Are Now Upgrade-Friendly**

* **New Feature**: `pg_dump` can now include optimizer stats.
* **Command**: `pg_dump --with-statistics`
* **Caveat**: Extended stats not included yet — run:

  ```bash
  vacuumdb --all --analyze-only --analyze-in-stages
  ```

  post-upgrade to patch missing stats.

---

## 🔥 7. **How Someone Accidentally Nuked Their Prod DB**

* **Setup**: Cascade delete + row-level security.
* **Mistake**: Deleted a `user` record to "fix" a problem — FK cascade wiped *all related data*.
* **Fallout**: 22 hours of data lost.
* **Recovery**: Paid plan + backfill via logs.
* **Lesson**: Never delete user records blindly. Always know your FK + cascade behavior.

---

## 📈 8. **Why P99 > Mean (Query Performance Analysis)**

* **P99**: Measures worst-case latency (slowest 1% of queries).
* **Why it matters**: Averages lie. P99 exposes spikes and outliers.
* **Tools**:

  * `pg_stat_statements` (no P99 out of the box)
  * `pg_stat_monitor` (P99 support, but may have perf overhead)
  * Suggestion: histograms for query timing would be ideal

---

## 🎥 9. **PGConf.dev Videos Posted**

* Most talks now live on their [YouTube channel](https://www.youtube.com) (not linked in transcript, just mentioned).

---

## 🚦 10. **Blue-Green Upgrades in AWS RDS: Gotchas**

* **Pros**:

  * Fast cluster creation via copy-on-write
  * Automated switchover
* **Cons**:

  * Poor rollback strategy for Postgres
  * Config mismatches trigger **forced reboots**
  * Must pre-tune green cluster params (e.g., for logical replication)
  * Best for fresh deploys, not flexible upgrades

---


