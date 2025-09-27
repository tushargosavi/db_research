# Logical Plan

## Overview
DataFusion represents logical plans using the `LogicalPlan` enum defined in `datafusion-optimizer`.
Plans are immutable trees built via builder APIs and SQL translation, enabling rule-based optimization
before conversion to physical plans.

## Node Types
- `Projection`, `Filter`, `Aggregate`, `Join`, `Sort`, `Limit`, `Repartition`, `Union`, `Distinct`, `Window`, and `Explain`.
- `TableScan` nodes wrap `TableSource` metadata including projection pushdown, filters, and statistics.
- `Subquery` nodes handle CTEs, scalar subqueries, and derived tables.

## Representation Patterns
- The enum-based representation leverages Rust's pattern matching for rule definitions.
- Expressions use `Expr` enum with variants like `Column`, `Literal`, `BinaryExpr`, `Case`, and `AggregateFunction`.
- Helper builders (`LogicalPlanBuilder`) provide fluent APIs to compose plans from DataFrame operations.

## Translation Flow
1. SQL parsed into AST via `sqlparser-rs`.
2. `SqlToRel` converts statements to `LogicalPlan` using context to resolve schemas and functions.
3. DataFrame API constructs plans directly (e.g., `df.filter(col("x").gt(lit(1)))`).

## Rewrite Rules
- Rule list managed by `Optimizer` includes passes for projection pushdown, predicate pushdown, constant folding,
  type coercion, filter reordering, and join reordering heuristics.
- Rules implement `OptimizerRule` trait with `optimize(&self, plan, state)` returning transformed plan.
- Applying a rule requires matching plan patterns using tree visitor functions and rewriting subtrees.

## Adding a New Rule
1. Implement `OptimizerRule` in a new module (e.g., `optimizer/src/rules`).
2. Use helper macros like `plan_rewrite` or manual recursion to locate target patterns.
3. Register the rule in `Optimizer::new()` sequence, placing it relative to dependent transformations.
4. Add unit tests under `optimizer/src/rules/tests` verifying canonical inputs and outputs.

## Expression Simplification
- Constant folding occurs in both logical expressions (`simplify_expressions` rule) and aggregator contexts.
- `ExprSimplifier` uses data type information from schemas to reduce arithmetic and boolean expressions.

## Improvement Opportunities
- Explore memoized plan representation (e.g., Cascades or DPhyp) for cost-based optimization.
- Extend statistics propagation to improve selectivity estimates feeding join reordering decisions.
