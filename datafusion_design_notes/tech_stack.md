# Tech Stack

## Overview
DataFusion is implemented primarily in Rust and leverages Apache Arrow as its in-memory columnar format.
The project ships both a core query engine and optional distributed runtime (Ballista) with consistent data
structures and planner components.

## Programming Languages and Core Libraries
- **Rust** for all core modules, leveraging traits, enums, and async features for type safety and performance.
- **Apache Arrow Rust crate** (`arrow` and `arrow-array`) for columnar buffers, array kernels, and IPC support.
- **Arrow Flight** (`arrow-flight`) to provide gRPC-based transport for distributed execution scenarios.
- **Parquet** (`parquet` crate) for columnar file read/write integration.

## Build and Packaging
- Built with **Cargo**, with workspaces dividing crates such as `datafusion-common`, `datafusion-expr`, `datafusion-optimizer`,
  and `datafusion-physical-plan` for modular compilation.
- Integration tests orchestrated via `cargo test` across workspace members; optional features enable subcomponents like
  DataFrame API, SQL planner, or Ballista runtime.
- Continuous integration uses GitHub Actions to run Rust unit tests, fmt, clippy, and integration pipelines.

## External Dependencies
- Depends on **Tokio** for async runtime primitives used by executors, data sources, and Ballista scheduler.
- Utilizes **prost** and **tonic** through Arrow Flight for protocol buffers and gRPC communication.
- Optional integration with **object store** crates (e.g., `object_store`) for cloud storage abstraction.
- Supports Substrait serialization via the `datafusion-substrait` crate for interoperability.

## Versioning and Compatibility
- Tracks Arrow releases closely; Cargo features gate compatibility with specific Arrow versions.
- Semantic versioning adopted with `datafusion` crates publishing under the Apache Software Foundation release process.
- Stability guarantees prioritize logical/physical plan APIs while allowing experimental features (e.g., async functions,
  `TableProvider` extensions) behind feature flags.

## Opportunities
- Evaluate embedding MLIR or other IR frameworks for code generation to improve performance-critical kernels.
- Explore `no_std` compatibility for embedded deployments by isolating OS-dependent components (e.g., object store, networking).
