# Optimizer

## Overview
DataFusion ships with a primarily rule-based optimizer that performs logical rewrites and limited cost-based
choices. The optimizer is extensible via pluggable rules and statistics providers.

## Rule-Based Optimization
- Rules implement the `OptimizerRule` trait and are executed in a configurable pipeline.
- Key rules: projection pruning, predicate pushdown, filter simplification, constant folding, type coercion,
  limit pushdown, `IN` list simplification, and join reorder heuristics.
- Pattern matching utilities (e.g., `OptimizerUtils::extract_columns`) help rules interact with expressions.

## Cost-Based Elements
- Statistics derived from `TableProvider::statistics` feed into `JoinSelection` rule to choose join algorithm
  (hash vs. nested loop) and order.
- Column statistics (min/max, null count) used for selectivity estimation in predicate pushdown and pruning.
- `OptimizerConfig` allows enabling cost-based decisions selectively (e.g., for distributed contexts).

## Statistics Collection
- `StatisticsProvider` trait fetches table-level and column-level metrics.
- Parquet reader extracts metadata (row groups, min/max) and surfaces through statistics to the optimizer.
- Users can plug custom providers for external catalogs; caching recommended for expensive metadata retrieval.

## Join Reordering
- Default heuristic uses commutativity/associativity rewrites to explore bushy trees limited by plan size.
- Cost-based reordering uses greedy algorithms, evaluating join selectivity from stats to choose best pairings.
- Support exists for broadcast hints via config or SQL hints to influence join type selection.

## Predicate Pushdown & Projection Pruning
- Performed across TableScan, Projection, and Aggregate nodes.
- Table providers expose `supports_filters_pushdown` to indicate supported filter types (exact, inexact, unsupported).
- Projection pruning uses expression analysis to compute required columns, reducing I/O and compute.

## Improvement Opportunities
- Introduce a memo-based optimizer (e.g., Cascades) to unify rule application and cost estimation.
- Expand statistics to include histograms and ndv estimates for more accurate selectivity.
- Provide adaptive runtime feedback loops to refine optimizer decisions based on observed execution metrics.
