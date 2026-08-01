# spark-rust: a Rust-native Spark engine that's embeddable and distributed

spark-rust is a Rust-native Spark SQL/DataFrame and Spark Connect engine. The
same Arrow-native core runs embedded in Rust or Python, as a Spark Connect
service, across a distributed executor cluster, and entirely in a browser via
WebAssembly.

[Try Spark SQL in your browser](https://sparksql-in-browser.sparkrust.org/) ·
[PySpark tutorial](https://pyspark-tutorial.sparkrust.org/) ·
[SQL tutorial](https://spark-sql-tutorial.sparkrust.org/) ·
[GitHub](https://github.com/tomz/spark-rust)

---

## One engine, four deployment shapes

### Embedded Rust or Python

Run the engine in-process with no JVM, daemon, or network hop. Use native Rust
DataFrames and typed Arrow results, or the embedded Python API for SQL,
DataFrames, Arrow, and pandas workflows.

### Single-node Spark Connect

Run the native `spark-connect-server` and connect with PySpark or the project’s
Rust, Java, Scala, SparkR-compatible, TypeScript, and Ruby clients.

### Distributed cluster

Use the same SQL and DataFrame plans across native executors with Arrow IPC
shuffle, spill, adaptive execution, executor-loss shuffle recovery, and
query-level retries.

### Browser WebAssembly

Run Spark-dialect SQL against Arrow data entirely inside a browser tab. No
server or data upload is required.

---

## Language APIs

The relational engine is available through:

- **Rust** — embedded `SparkSession` or remote `SparkConnectSession`, typed row
  decoding, Arrow results, and local lazy `Rdd<T>` lineage;
- **Python and PySpark** — embedded `sparkrust` or unmodified PySpark 3.5/4.x
  over Spark Connect;
- **Java and Scala 2.13** — Spark-shaped DataFrame and typed relational Dataset
  clients over Connect;
- **SparkR-compatible R** — SQL, DataFrames, grouping, joins, actions, local
  Arrow upload, and writes;
- **TypeScript/Node.js and Ruby** — native Connect clients;
- **SQL** — embedded, CLI, Spark Connect, HiveServer2/JDBC/ODBC, and browser
  execution.

```rust
use spark_rust::prelude::*;

#[tokio::main]
async fn main() -> spark_rust::Result<()> {
    let spark = SparkSession::builder().app_name("sales").build();
    let rows = spark
        .sql("SELECT * FROM VALUES ('west', 2), ('west', 8), ('east', 4) AS sales(region, amount)")
        .await?
        .filter(col("amount").geq(lit(3_i64)))?
        .group_by([col("region")])
        .agg([sum(col("amount")).alias("total")])?
        .order_by([col("total").desc()])?
        .collect_as::<(String, i64)>()
        .await?;
    println!("{rows:?}");
    Ok(())
}
```

The same three workloads — filter → group → sum → order; customer/order join →
group → sum → order; null filtering → distinct → descending order → limit —
are implemented in the Rust, Python, Java, Scala, SparkR-compatible R,
TypeScript/Node, and SQL APIs, and produce identical results in each.

---

## Major capabilities

- **Spark SQL and DataFrames** — projections, filters, joins, aggregates,
  windows, grouping sets, `ROLLUP`, `CUBE`, CTEs, generators, DDL/DML, complex
  types, intervals, timestamps, JSON/variant, sketches, and Spark-specific
  functions.
- **T-SQL frontend** — an opt-in, AST-aware hybrid transpiler built on the
  Microsoft SQL Server grammar. It supports bracket identifiers, `TOP`,
  `OFFSET/FETCH`, `ISNULL`, `IIF`, style-free `CONVERT`/`TRY_CONVERT`, date
  functions, and grouping constructs while retaining the normal Spark/DataFusion
  optimizer and Arrow execution engine. See
  [the T-SQL design and readiness guide](docs/tsql-support-readiness-and-design.md).
- **Spark compatibility** — a broad regression harness continuously exercises
  Spark SQL syntax, functions, schemas, errors, DataFrame behavior, and Connect
  protocol compatibility.
- **Spark Connect** — SQL, DataFrames, actions, writes, catalogs, ML,
  Structured Streaming, Python UDFs, auth, TLS, cancellation, reattachment,
  and graceful drain.
- **HiveServer2/JDBC/ODBC** — a real Thrift `TCLIService` with binary and HTTP
  transports, optional TLS, and SASL/PLAIN.
- **Adaptive execution** — partition coalescing, skew splitting, runtime
  broadcast selection, join strategy switching, runtime filters, and stage
  statistics.
- **Scheduling and recovery** — distributed SQL dispatch, bounded retries,
  executor decommissioning, External Shuffle Service ownership transfer, query
  replay, and explicit failure handling.
- **Shuffle and spill** — gRPC/Arrow shuffle, External Shuffle Service,
  local/ESS reducer merge with pull fallback, and spillable sort, aggregate,
  join, window, cross-join, and shuffle paths.
- **Structured Streaming** — micro-batch, continuous, and resident Real-Time
  runtimes; watermarks, event-time windows, state stores, checkpoints,
  Kafka/socket/rate/file sources, file/Parquet/Kafka sinks, stateful operators,
  lifecycle management, and live progress reporting.
- **MLlib and native ML** — broad Spark ML estimator/model coverage, held-out
  CV/TVS, distributed multi-feature OLS moments, TreeSHAP, conformal prediction,
  HNSW, causal learners, forecasting, drift/fairness, and AutoML-style search.
- **Lakehouse federation** — Delta Lake with Liquid Clustering, Iceberg, Hudi, Hive Metastore, Fabric OneLake, Databricks Unity Catalog, and cross-catalog joins in one query.
- **Formats and storage** — Parquet, ORC, CSV, JSON, Avro, Arrow IPC, local
  filesystems, S3-compatible stores, Azure/OneLake, GCS, HDFS, and HTTP-backed
  object stores.
- **GPU acceleration** — optional CUDA kernels and an optional cuDF Parquet
  path, both retaining automatic CPU fallback.
- **Operations** — live topology dashboard, event logs, History Server,
  Prometheus metrics, Kubernetes/Helm/operator resources, graceful executor
  decommissioning, and dynamic-allocation controls.

---

## Live browser demos

**SQL**

- [Spark SQL playground](https://sparksql-in-browser.sparkrust.org/)
- [Spark SQL tutorial](https://spark-sql-tutorial.sparkrust.org/)

**Language APIs**

- [PySpark playground](https://pyspark-in-browser.sparkrust.org/)
- [PySpark tutorial](https://pyspark-tutorial.sparkrust.org/)
- [Rust API playground](https://rust-in-browser.sparkrust.org/)
- [Scala API playground](https://scala-in-browser.sparkrust.org/)
- [SparkR API playground](https://sparkr-in-browser.sparkrust.org/)
- [Ruby API playground](https://ruby-in-browser.sparkrust.org/)
- [TypeScript API playground](https://spark-in-browser.sparkrust.org/)

**Workloads and analytics**

- [TPC-DS explorer](https://tpcds-in-browser.sparkrust.org/)
- [TPC-H explorer](https://tpch-in-browser.sparkrust.org/)
- [BI dashboard](https://bi-in-browser.sparkrust.org/)
- [What-if simulator](https://whatif-in-browser.sparkrust.org/)
- [Structured Streaming](https://streaming-in-browser.sparkrust.org/)
- [ML classification](https://sparkml-in-browser.sparkrust.org/)
- [Geospatial queries](https://sparkspatial-in-browser.sparkrust.org/)

**Operations**

- [Mission control](https://demo.sparkrust.org/)

The same SQL compatibility core and Spark UDF registry are compiled to
WebAssembly and run against in-memory Arrow batches in a Web Worker.

---

## Performance positioning

spark-rust is designed to be competitive with **DuckDB** for embedded,
in-process, and single-node analytical workloads, and competitive with
**Apache Spark** when the same engine is deployed across a distributed
executor cluster.

The Arrow-native execution core, cost-based and adaptive optimizers, spillable
operators, runtime filtering, native shuffle, and shared SQL/DataFrame semantics
apply across both deployment modes. This lets teams prototype locally and scale
the same workloads out without moving to a different query engine or rewriting
the application.

---

## Get started

No install required — the engine runs in your browser:

- [Spark SQL console](https://sparksql-in-browser.sparkrust.org/)
- [PySpark tutorial](https://pyspark-tutorial.sparkrust.org/)
- [Spark SQL tutorial](https://spark-sql-tutorial.sparkrust.org/)
- [Structured Streaming](https://streaming-in-browser.sparkrust.org/)
- [MLlib classification](https://sparkml-in-browser.sparkrust.org/)
- [Geospatial](https://sparkspatial-in-browser.sparkrust.org/)
- [TPC-DS](https://tpcds-in-browser.sparkrust.org/) ·
  [TPC-H](https://tpch-in-browser.sparkrust.org/) ·
  [BI dashboard](https://bi-in-browser.sparkrust.org/)
- Language APIs:
  [Rust](https://rust-in-browser.sparkrust.org/) ·
  [Python](https://pyspark-in-browser.sparkrust.org/) ·
  [Scala](https://scala-in-browser.sparkrust.org/) ·
  [R](https://sparkr-in-browser.sparkrust.org/) ·
  [TypeScript](https://spark-in-browser.sparkrust.org/) ·
  [Ruby](https://ruby-in-browser.sparkrust.org/)

### Run the native playground with Docker

Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) or
[Docker Engine](https://docs.docker.com/engine/install/), then run the
recommended binary-only edition:

```bash
docker pull ghcr.io/tomz/sparkrust-playground:medium-v0.42.1

docker run --rm --name sparkrust-playground \
  --memory=12g --memory-swap=12g \
  -p 127.0.0.1:8000:8000 \
  ghcr.io/tomz/sparkrust-playground:medium-v0.42.1
```

Open <http://127.0.0.1:8000> and sign in with username `sparkrust` and password
`sparkrust`. The multi-platform image automatically selects native
`linux/amd64` or `linux/arm64` layers.

Choose an edition based on the notebook kernels you need:

| Edition | Image tag | Memory | Included kernels |
|---|---|---:|---|
| **Medium — recommended** | `medium-v0.42.1` | 12 GiB | Python, Spark SQL, Scala, SparkR, TypeScript, Ruby |
| **Large — full developer SDK** | `large-v0.42.1` or `v0.42.1` | 20 GiB | Medium plus full and instant Rust kernels and JavaScript |
| **Small — minimal trial** | `small-v0.42.1` | 8 GiB | Python/PySpark and Spark SQL |

Replace the image tag and memory limit in the command above to run another
edition. The unqualified `latest` tag tracks the large edition; pin `v0.42.1`
or an edition-qualified tag for reproducible use.

> **For most users: keep it local.** Use the command above exactly as written,
> leave the port bound to `127.0.0.1`, and open `http://127.0.0.1:8000` on the
> same computer. Do not change the binding to `0.0.0.0` and do not forward port
> `8000` from your router.

To use a playground running on a remote server, keep its Docker port private and
open an SSH tunnel from your laptop:

```bash
# Run this on the remote server. Port 8000 remains server-local.
docker run --rm --name sparkrust-playground \
  --memory=12g --memory-swap=12g \
  -p 127.0.0.1:8000:8000 \
  ghcr.io/tomz/sparkrust-playground:medium-v0.42.1

# Run this in a second terminal on your laptop.
ssh -N -L 8000:127.0.0.1:8000 YOUR_USER@YOUR_SERVER
```

Then open <http://127.0.0.1:8000> on your laptop. The browser traffic travels
inside SSH; the Hub is still not exposed to the Internet. Stop the tunnel with
`Ctrl-C`.

The bundled Hub is a single-user trial with a shared password, not a production
multi-user service. If multiple untrusted users need access, do not publish this
container directly. Use a production JupyterHub deployment with per-user
OIDC/SSO, an access policy, and HTTPS.

For local testing, credentials can be changed with
`-e JUPYTERHUB_USERNAME=... -e JUPYTERHUB_PASSWORD=...`. JupyterLab uses
container port `8888`, Spark Connect uses `15002`, and the large and medium
editions also provide federated Connect on `15003`; leave those ports unpublished
unless you specifically need them.

---

spark-rust is built on
[Apache DataFusion](https://datafusion.apache.org/) and
[Apache Arrow](https://arrow.apache.org/), and targets compatibility with
the [Apache Spark](https://spark.apache.org/) API surface.
