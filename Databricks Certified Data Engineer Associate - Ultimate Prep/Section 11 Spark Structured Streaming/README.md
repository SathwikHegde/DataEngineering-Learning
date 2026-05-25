# Section 11: Spark Structured Streaming

This section introduces **Spark Structured Streaming**, the scalable and fault-tolerant stream processing engine built on the Spark SQL engine. In 2026, the focus has shifted toward high-performance, low-latency "Lakeflow" patterns and advanced recovery mechanisms. Mastering these four modules is essential for handling real-time data ingestion and processing in a production Lakehouse environment.

---

## Section Modules

### 76. Structured Streaming - Introduction (6 min)

* **The Programming Model**: Treat a live data stream as a table that is being continuously appended. This allows you to express your streaming computation the same way you would a batch computation.
* **Key Components**:
* **Source**: Where data originates (Kafka, Cloud Files/Auto Loader, Delta Tables).
* **Sink**: Where processed data is written (Delta Lake is the industry standard in 2026).


* **Stateless vs. Stateful**: Understanding when Spark needs to maintain intermediate state (like windowed aggregations) versus simply transforming data line-by-line.

### 77. Structured Streaming - Demo (16 min)

* **Hands-on Workflow**: A live walkthrough of reading from a streaming source and writing to a Delta table.
* **2026 Update**: Introduction to **Lakeflow Spark Declarative Pipelines**, which simplify the management of these streams by automating the underlying infrastructure.
* **Resource Check**: Access the `Resources` dropdown in the course player to download the specific notebooks used for this demonstration.

### 78. Trigger & OutputMode (4 min)

* **Trigger Modes (2026 Standards)**:
* **Unspecified (Default)**: Micro-batches are processed as soon as the previous one finishes.
* **ProcessingTime**: Runs micro-batches at fixed intervals (e.g., `trigger(processingTime='1 minute')`).
* **AvailableNow**: Processes all available data as a batch and then stops. **This is the Databricks recommendation for serverless compute in 2026.**
* **Real-time Mode (Preview)**: A new 2026 feature for sub-second, ultra-low latency (down to 5ms) for operational use cases like fraud detection.


* **Output Modes**:
* **Append**: Only new rows added to the Result Table are written to the sink (default).
* **Complete**: The entire Result Table is rewritten to the sink (used for aggregations).
* **Update**: Only the rows that were updated in the Result Table are written (useful for Delta tables).



### 79. Checkpointing (3 min)

* **The Foundation of Fault Tolerance**: A checkpoint stores the current state and offsets of your stream in cloud storage (DBFS or Unity Catalog Volumes).
* **Exactly-Once Guarantees**: By using a unique `checkpointLocation` for every stream, Spark can recover from failures and pick up exactly where it left off without duplicating or losing data.
* **2026 Best Practice**: Always use **Asynchronous State Checkpointing** for stateful queries to minimize micro-batch latency.

---

## Performance Checklist for 2026

* **Never Use All-Purpose Compute**: In production, always run Structured Streaming workloads using **Jobs Compute** to save costs and ensure dedicated resources.
* **Disable Autoscaling**: For standard streaming jobs, autoscaling can cause instability. If you need scaling, migrate to **Lakeflow Pipelines** which feature enhanced, streaming-aware autoscaling.
* **Watermarking**: Remember that if you are doing stateful aggregations (like "orders per hour"), you must define a **Watermark** to handle late-arriving data and prevent the state from growing infinitely.

---

[Next Section: Delta Live Tables & Production Pipelines →](https://www.google.com/search?q=./section12-readme.md)

Are you planning to run these streams continuously for real-time dashboards, or are you looking to use the `AvailableNow` trigger for scheduled incremental processing?