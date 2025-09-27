# Monitoring and Diagnostics

## Overview
Observability features in DataFusion provide insight into query execution through logging, metrics, and
plan explainers. These tools aid performance tuning and troubleshooting.

## Logging
- Uses Rust `log` facade with env_logger or tracing subscribers for structured output.
- Execution events logged at debug level include plan creation, optimizer passes, and operator scheduling.
- Ballista extends logging with job lifecycle events and executor status updates.

## Metrics
- `MetricSet` attached to `ExecutionPlan` nodes collects counters (input/output rows, batches), gauges (peak memory),
  and timings (elapsed compute). Metrics aggregated per partition and exposed via `ExecutionMetrics`.
- Optional integration with `metrics` crate enables exporting to Prometheus or other backends.
- Ballista exposes REST/gRPC endpoints for cluster-wide metrics collection.

## Explain and Analyze
- `EXPLAIN` statement returns logical and physical plan text along with optimizer rule application traces.
- `EXPLAIN ANALYZE` executes the query and returns collected metrics embedded in the plan tree.
- JSON representation of plans available via `display::plan` utilities for programmatic consumption.

## Profiling and Debugging
- Feature flags enable tracing with `tokio-tracing` to capture spans per operator.
- Integration with `pprof` crate allows CPU and heap profiling snapshots.
- Unit and integration tests include `assert_batches_eq!` macros to validate execution correctness.

## Improvement Opportunities
- Provide query timeline visualization aggregating metrics over time for long-running queries.
- Add event-driven alerts for skew detection or repeated spills to disk.
