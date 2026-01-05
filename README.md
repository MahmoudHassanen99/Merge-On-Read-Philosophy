# Storage Strategy Philosophy: CoW vs. MoR

This documentation explores the fundamental trade-offs between **Copy-on-Write (CoW)** and **Merge-on-Read (MoR)** storage strategies. These patterns define how modern data engines like **Apache Iceberg** and **ClickHouse** manage updates and deletes in large-scale analytical environments.

---

**Explore the Interactive Simulation:** [merge-on-read-philosophy.vercel.app](https://merge-on-read-philosophy.vercel.app/)

---

## 🏛 The Core Dilemma
At the heart of data engineering lies a constant tension: **Do we pay the performance cost during the Write operation or the Read operation?**

### 1. Copy-on-Write (CoW)
**"Pay Now, Save Later"**

In a CoW system, whenever a record is updated or deleted, the engine identifies the file containing that record and **rewrites the entire file** into a new version.

* **Pros:** Data is physically contiguous and optimized on disk. Queries are extremely fast because there is no computation required to "reconcile" changes at runtime.
* **Cons:** High **Write Amplification**. Updating a single 1KB row in a 1GB file requires rewriting the full 1GB.
* **Best Use Case:** Batch processing, high-frequency reads, and stable historical data.

### 2. Merge-on-Read (MoR)
**"Save Now, Pay Later"**

In an MoR system, the engine does not touch the original data files. Instead, it writes a small **"delta" or "delete file"** that records the change. 

* **Pros:** Extremely low ingestion latency. High-velocity streaming data can be committed almost instantly.
* **Cons:** High **Read Amplification**. The query engine must perform a "merge" operation at runtime, joining the base data with all existing delta files to present the correct state.
* **Best Use Case:** Real-time Change Data Capture (CDC), streaming pipelines, and high-frequency update workloads.

---

## 🔄 Engine Comparison

How the two philosophies manifest in popular data platform tools:

| Feature | Apache Iceberg | ClickHouse |
| :--- | :--- | :--- |
| **CoW Implementation** | Rewrites full Parquet files. Ideal for Spark/Flink batch jobs. | Handled via `ALTER TABLE ... UPDATE` (mutations). |
| **MoR Implementation** | Uses **Delete Files** (Position or Equality). | Uses **Patch Parts** (Deltas) merged during `FINAL` queries. |
| **Primary Goal** | Ensuring consistent snapshot isolation in Data Lakes. | Achieving sub-second query performance on massive datasets. |

---

## 📊 Summary Table

| Metric | Copy-on-Write (CoW) | Merge-on-Read (MoR) |
| :--- | :--- | :--- |
| **Write Performance** | Low (Heavy) | High (Light) |
| **Read Performance** | High (Optimal) | Lower (Computationally expensive) |
| **Storage Layout** | Clean / Optimized | Fragmented (Base + Deltas) |
| **Best Workflow** | Data Warehousing / Dashboards | Real-time Analytics / CDC |
