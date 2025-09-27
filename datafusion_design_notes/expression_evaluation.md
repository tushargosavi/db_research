# Expression Evaluation

## Overview
Expression evaluation underpins DataFusion's performance. Logical expressions (`Expr`) are compiled into
`PhysicalExpr` implementations executed against Arrow `RecordBatch`es. The engine emphasizes vectorized
interpretation with optional code generation for Substrait/externals, balancing flexibility and speed.

## Expression AST Structure
- Logical expressions modeled via `Expr` enum in `datafusion-expr`, covering columns, literals, binary/ternary ops,
  scalar/aggregate/window functions, casts, case expressions, and subqueries.
- Physical expressions implement `PhysicalExpr` trait with `evaluate(&self, batch)` returning `ColumnarValue`.
- Visitor utilities like `ExprVisitor` enable traversing expressions for analysis (e.g., column collection).

## Type System and Coercion
- `DataType` enumeration from Arrow defines supported types; DataFusion extends semantics via `DataFusionError`
  for unsupported casts or operations.
- `TypeCoercion` module computes common supertypes for binary operators, aligning with SQL rules (numeric promotion,
  string concatenation, boolean logic).
- Implicit casting occurs during logical planning; runtime evaluation uses explicit `CastExpr` nodes.

## Operators and Extensibility
- Operators enumerated in `Operator` (arithmetic, comparison, boolean). Each maps to compute kernels in Arrow.
- New operators require extending `Operator` enum, implementing physical evaluation using Arrow kernels, and updating
  planner coercion logic.
- Complex expressions (e.g., `IN`, `LIKE`, regex) implemented via specialized `ScalarFunctionExpr` wrappers.

## Type Casting and Error Handling
- `CastExpr` checks validity and uses Arrow's `compute::cast` kernels; errors propagate as `DataFusionError::Execution`.
- Overflow detection depends on Arrow kernels (e.g., `checked_add`) to return errors when overflow occurs.
- Division by zero results in Arrow errors; DataFusion surfaces them to the query engine for proper handling.

## Evaluation Modes
- **Vectorized interpretation**: Default mode; each `PhysicalExpr` evaluates entire batch via Arrow kernels.
- **Scalar fallback**: For expressions lacking vectorized kernels, DataFusion iterates row-wise using `ScalarValue` utilities.
- **Code generation**: Experimental via Substrait and external projects (e.g., using LLVM) to compile expressions into
  native code for specific targets.

## Vectorized Details
- Batch size configurable (default 8192). `ColumnarValue` returns either an array or scalar broadcast.
- SIMD usage delegated to Arrow compute kernels; DataFusion benefits automatically when compiled with CPU feature flags.
- Loop unrolling achieved within Arrow kernels; DataFusion focuses on minimizing per-row dispatch overhead.

## Scalar Function Evaluation
- Built-in scalar functions registered via `ScalarUDF`. Each has a `Signature`, `ReturnTypeFunction`, and `ScalarFunctionImplementation`.
- Implementations typically call Arrow kernels; for complex cases (e.g., regex), rely on `regex` crate compiled iterators.
- UDFs can be registered at runtime by providing closures conforming to `ScalarUDF` API.

## Aggregate Function Evaluation
- Aggregate expressions represented by `AggregateExpr` trait implementations (`Sum`, `Avg`, `Min`, `Max`, `ApproxDistinct`).
- Partial aggregation uses accumulator structs implementing `Accumulator` trait with `update_batch`, `merge_batch`, `evaluate`.
- Hash aggregate organizes accumulators per group key; supports two-phase (partial/final) execution for distributed contexts.

## Window Function Evaluation
- Implemented via `WindowAggExec` combining partitioning, ordering, and frame specifications.
- Uses sliding accumulators and partition-boundary detection to compute window results efficiently.
- Supports row-based and range-based frames; ongoing work extends to groups-based frames.

## User-Defined Functions
- Scalar UDFs registered through `SessionContext::register_udf`. DataFusion ensures type checking using signatures.
- Aggregate UDFs implement `AggregateUDF` with custom accumulators; state serialization supports distributed merges.
- User-Defined Table Functions (UDTF) emerging via `TableFunction` trait for generating relation-valued outputs.

## Optimization Techniques
- `SimplifyExpressions` rule performs constant folding, boolean simplification, and canonicalization.
- Common subexpression elimination supported via `ExprRewriter` visitors caching evaluation results across projections.
- `Repartition` and `CoalesceBatches` operators used to align batch sizes and reduce redundant evaluations.

## Error Handling
- Evaluation errors propagate as `DataFusionError::Execution`, captured per task and aggregated at query level.
- Configurable to continue on error for some data sources (e.g., `CsvOptions::skip_invalid_rows`).
- Null semantics preserved using Arrow's three-valued logic across logical and physical layers.

## Performance Considerations
- Benchmarks show vectorized evaluation achieving millions of rows/sec on modern CPUs due to Arrow kernels.
- Bottlenecks include dynamic dispatch overhead for `PhysicalExpr` trait objects and limited code generation.
- Proposed improvements: expression fusion to reduce kernel launches, caching compiled expressions via JIT, and
  integrating hardware-specific kernels (AVX-512, GPU) through Arrow extensions.
