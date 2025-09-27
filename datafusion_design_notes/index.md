# DataFusion Design Notes

This collection summarizes key architectural components of Apache DataFusion, a Rust-based query execution
framework built on Apache Arrow. Each section distills implementation details, design trade-offs, and improvement
ideas derived from the upstream project, focusing particularly on the expression evaluation subsystem.

## Component Guide

- [Tech Stack](tech_stack.md)
- [SQL Frontend](sql_frontend.md)
- [Metadata Management](metadata_management.md)
- [Logical Plan](logical_plan.md)
- [Optimizer](optimizer.md)
- [Physical Plan](physical_plan.md)
- [Memory Layout](memory_layout.md)
- [Expression Evaluation](expression_evaluation.md)
- [Execution Engine](execution_engine.md)
- [Runtime](runtime.md)
- [Storage Integration](storage_integration.md)
- [Monitoring and Diagnostics](monitoring_diagnostics.md)

## Research Notes

- Expression evaluation is the primary performance lever in DataFusion. The subsystem balances Arrow-native
  vectorized kernels with extensible UDFs, while exploring optional code generation via the DataFusion Substrait
  bridge and potential LLVM integration.
- Optimizer capabilities have evolved from purely rule-based rewrites toward selective cost-based decisions using
  statistics providers, though the system still leans heavily on heuristics.
- Distributed execution through Ballista shares many components with core DataFusion but introduces scheduling and
  shuffle services that can inspire future runtime work.
