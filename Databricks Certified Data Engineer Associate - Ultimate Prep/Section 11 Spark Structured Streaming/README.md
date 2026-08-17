# Section 11: Spark Structured Streaming

This section introduces **Spark Structured Streaming**, the scalable, fault-tolerant stream processing engine built natively on the Spark SQL optimization core. Production focus centers on high-performance, low-latency streaming patterns, Lakeflow pipeline integration, and resilient asynchronous state recovery mechanisms. Mastering these operational modules is essential for executing real-time data ingestion and event processing inside an enterprise Lakehouse environment.

Refer to your course dashboard for the corresponding lesson timeline and visual assets.

---

## Section Overview

* **Total Duration:** 29 minutes
* **Total Lessons:** 4
* **Primary Focus:** Unbounded append table abstractions, micro-batch trigger optimization, fault-tolerant state boundaries, and production operational guardrails.

---

## Curriculum Breakdown

### 76. Structured Streaming — Introduction (6 min)

* **The Unbounded Table Model**: Treating live, incoming data streams as unbounded append-only tables. This paradigm allows streaming transformations and aggregations to use the exact same declarative DataFrame API and SQL syntax as static batch queries.
* **Core Stream Primitives**:
* **Source**: The streaming ingest interface (e.g., Apache Kafka, Cloud Files/Auto Loader, upstream Delta tables).
* **Sink**: The target destination storage layer (Delta Lake serves as the ACID-compliant standard).


* **Stateless vs. Stateful Processing**:
* **Stateless Transformations**: Record-by-record operations evaluated independently of previous rows (e.g., `select()`, `filter()`, `withColumn()`).
* **Stateful Transformations**: Historical cross-row state tracking across micro-batch boundaries (e.g., windowed aggregations, stream-stream joins, deduplication).



### 77. Structured Streaming — Demo (16 min)

* **Hands-on Stream Orchestration**: Live implementation of reading unstructured streaming telemetry from cloud storage and writing it statefully into a Delta table sink.
* **Lakeflow Integration**: Deploying stream definitions within **Lakeflow Declarative Pipelines (DLT)** to automate infrastructure provisioning, cluster maintenance, and checkpoint lifecycle management.
* **Notebook Resources**: Access the `Resources` dropdown in the course player to download the accompanying PySpark and Spark SQL demonstration notebooks.

### 78. Trigger & OutputMode (4 min)

* **Trigger Execution Mechanics**:
* **Unspecified (Default)**: Micro-batches execute continuously and sequentially as soon as the previous micro-batch completes.
* **ProcessingTime**: Restricts execution evaluations to fixed intervals (e.g., `.trigger(processingTime='1 minute')`).
* **AvailableNow**: Ingests all available outstanding records incrementally across optimized micro-batches and terminates the compute session upon completion. **Recommended pattern for cost-efficient serverless orchestration.**
* **Real-time Mode**: Continuous low-latency processing framework designed for millisecond-level event handling.


* **Output Modes**:
* **Append (Default)**: Emits only net-new records inserted into the Result Table since the previous micro-batch.
* **Complete**: Re-computes and overwrites the entire Result Table state on every commit (mandatory for queries with streaming aggregations).
* **Update**: Emits only rows that have been updated or appended during the latest execution window.



### 79. Checkpointing (3 min)

* **Fault Tolerance Architecture**: Persisting streaming offsets, source metadata, and state store snapshots directly into cloud storage (or Unity Catalog Volumes).
* **Exactly-Once Processing**: Binding a unique `checkpointLocation` directory ensures that following cluster reboots or runtime failures, the query planner reconstructs the exact transaction log state and resumes without duplicate writes or data loss.
* **Asynchronous State Checkpointing**: Offloading state store serialization to background I/O threads, preventing state persistence from blocking active micro-batch computation cycles.

---

## Important Exam Considerations

* **Compute Unit Economics**: Production streaming jobs should run on dedicated **Job Compute** rather than interactive All-Purpose clusters to benefit from lower DBU billing rates and isolated resource footprints.
* **Elastic Scaling Stability**: Traditional cluster autoscaling can destabilize streaming queries due to node re-shuffling. Stateful workloads requiring dynamic scalability should use **Lakeflow Declarative Pipelines**, which support streaming-aware, load-balanced autoscaling.
* **Watermarking & State Eviction**: Stateful streaming aggregations require an explicit **Watermark** (e.g., `.withWatermark("event_time", "10 minutes")`) to establish late-arrival thresholds and purge stale state data from executor memory to prevent out-of-memory (`OOM`) crashes.

---

[← Back to Section 10: Advanced Transformations & Complex Data Structures](https://www.google.com/search?q=./section10-readme.md) | [Next Section: Section 12: Ingestion with Lakeflow Connect →](https://www.google.com/search?q=./section12-readme.md)
