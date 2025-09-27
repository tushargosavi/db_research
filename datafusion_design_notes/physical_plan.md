# Physical Plan

## Overview
Physical plans in DataFusion are represented by implementations of the `ExecutionPlan` trait defined in the
`datafusion-physical-plan` crate. Plans are vectorized operators that produce Arrow `RecordBatch` streams.

## Node Types
- **Scan Operators**: `ParquetExec`, `CsvExec`, `JsonExec`, `AvroExec`, `ListingTableExec`.
- **Relational Operators**: `ProjectionExec`, `FilterExec`, `HashAggregateExec`, `SortExec`, `WindowAggExec`,
  `LimitExec`, `CrossJoinExec`, `HashJoinExec`, `NestedLoopJoinExec`.
- **Utility Operators**: `CoalescePartitionsExec`, `RepartitionExec`, `MergeExec`, `ExplainExec`, `AnalyzeExec`.

## Representation
- Each plan implements `ExecutionPlan` trait methods: `schema()`, `output_partitioning()`, `children()`, and
  `execute(partition, context)` returning async stream of `RecordBatch`.
- Plans compose via trait objects (`Arc<dyn ExecutionPlan>`), enabling runtime polymorphism.
- `PhysicalExpr` trait mirrors logical expressions for evaluation on batches.

## Execution Model
- Operators follow a vectorized iterator pattern, pulling batches from children via async streams.
- Execution context orchestrated by `TaskContext` which carries runtime state, memory managers, and session configs.
- Partitioning metadata guides scheduler decisions: `Unbounded`, `Hash`, `RoundRobin`, `UnknownPartitioning`.

## Hardware Integration
- Arrow kernels provide SIMD-optimized primitives for arithmetic, comparisons, and aggregations.
- GPU integration is experimental; DataFusion relies on Arrow's device-agnostic buffers and is exploring partnerships
  with projects like Arrow CUDA or external accelerators.
- Supports spilling to disk via `DiskManager` when memory limits are hit during sort or aggregate operations.

## Logical-to-Physical Translation
- `PhysicalPlanner` trait transforms `LogicalPlan` into physical plans, invoking `DefaultPhysicalPlanner` by default.
- Table providers supply custom `create_physical_plan` to inject specialized operators.
- Physical expression planner (`create_physical_expr`) converts logical expressions respecting type coercion rules.

## Improvement Opportunities
- Adopt adaptive batch sizing based on downstream backpressure and Arrow compute kernel throughput.
- Explore asynchronous prefetch for file scans to overlap I/O and compute within executors.
