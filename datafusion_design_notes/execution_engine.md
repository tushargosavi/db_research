# Execution Engine

## Overview
The execution engine coordinates physical plan execution by scheduling `ExecutionPlan` partitions across threads.
It relies on the `TaskScheduler` and `RuntimeEnv` abstractions to manage resources and ensure efficient batch
processing.

## Processing Model
- **Vectorized, pull-based** execution: Operators request batches from children via async streams.
- Each partition executes independently, enabling embarrassingly parallel workloads.
- Supports both `collect()` (materialize in memory) and streaming results through `SendableRecordBatchStream`.

## Pipelining vs. Materialization
- Most operators pipeline batches directly; materialization occurs when required (e.g., `SortExec`, `HashAggregateExec` for finalization).
- `RepartitionExec` introduces shuffle boundaries, requiring data redistribution and potential buffering.

## Memory and Resource Management
- `RuntimeEnv` holds memory manager, disk manager, and object store clients.
- Spillable operators coordinate with `DiskManager` to avoid out-of-memory scenarios.
- Configurable concurrency via `SessionConfig::target_partitions` sets default partition count for parallelism.

## Task Scheduling
- Local execution uses a cooperative async runtime (Tokio) to poll operator futures.
- For distributed Ballista runtime, scheduler assigns tasks to executors using work-stealing and monitors progress via heartbeats.
- Fine-grained metrics collected per task to feed monitoring and adaptive behaviors.

## Fault Tolerance
- Single-node DataFusion relies on upstream error propagation; failures bubble to clients.
- Ballista introduces retry logic for failed tasks, rescheduling partitions as needed.

## Improvement Opportunities
- Introduce adaptive execution (dynamic repartitioning based on runtime stats).
- Explore integration with Arrow Flight SQL for streaming results with backpressure control.
