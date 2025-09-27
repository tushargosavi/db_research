# Runtime

## Overview
DataFusion's runtime environment (`RuntimeEnv`) encapsulates thread pools, memory management, disk spill
configuration, and object store clients. It powers both standalone execution and distributed Ballista deployments.

## Asynchronous Operations
- Built on **Tokio** async runtime for scheduling futures representing operator execution.
- File readers (Parquet, CSV) use async I/O when available; blocking tasks executed via dedicated thread pools to
  avoid stalling async reactor.
- Ballista leverages async gRPC (Tonic) for scheduler-executor communication.

## Operator Scheduling
- Single-node mode uses a task-per-partition model executed on a shared thread pool.
- Ballista scheduler assigns tasks to executors, employing work-stealing to balance load.
- Configurable concurrency via `RuntimeConfig::worker_threads` allows tuning per deployment.

## Parallelism Models
- Intra-query parallelism achieved by partitioning input scans and distributing across worker threads.
- Hash and sort aggregates support multi-stage execution (partial + final) to exploit partition-level parallelism.
- Ballista adds inter-node parallelism with shuffle stages persisted via object store or Flight.

## Distributed Execution
- Ballista's architecture includes scheduler, executors, and persistent state (etcd or memory) for tracking jobs.
- Data shuffling implemented with Arrow Flight or object store intermediates (e.g., S3), enabling fault tolerance.
- Supports task retries, speculative execution (configurable), and executor heartbeats for liveness.

## Improvement Opportunities
- Integrate adaptive resource scaling for elastic cloud environments based on workload intensity.
- Explore cooperative cancellation propagation across tasks to reduce wasted work on query aborts.
