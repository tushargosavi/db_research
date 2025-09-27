# Memory Layout

## Overview
DataFusion relies on Apache Arrow's columnar memory format, inheriting its contiguous buffers, validity bitmaps,
and offset arrays. Operators process data in batches of Arrow `RecordBatch` objects to maximize cache locality and
SIMD efficiency.

## Data Representation
- Columnar arrays store values contiguously per column; nested types use offset and value buffers.
- Scalar values represented via Arrow primitives (e.g., `Int64Array`, `Float64Array`). Strings use `BinaryArray` with
  offset buffers.
- Struct and list types supported through Arrow's nested array abstractions, enabling semi-structured data.

## Buffer Management
- Memory allocations handled via `arrow::buffer::Buffer` and `MutableBuffer`, backed by aligned allocations.
- `MemoryManager` in DataFusion enforces per-query and global memory limits; integrates with `RuntimeEnv`.
- Spill support uses `DiskManager` to offload oversized buffers for sort/aggregate operations.

## Null Handling
- Validity bitmaps accompany each array, enabling branch-free null checks via bitwise operations.
- Kernels propagate null semantics using Arrow's compute utilities, ensuring three-valued logic.
- `BooleanArray` specialized for bit-packed storage to reduce memory footprint.

## Compression and Encoding
- In-memory buffers are uncompressed; compression handled at storage layer (e.g., Parquet encoding).
- Dictionary encoding supported via `DictionaryArray` to reduce memory for low-cardinality columns.
- Run-length encoding handled within Parquet reader, expanded into Arrow arrays during execution.

## Vectorized Batch Processing
- Default batch size configurable (`config.batch_size`, typically 8192 rows).
- Operators iterate over `RecordBatch` streams, enabling vectorized compute kernels like `arrow::compute::add`.
- Evaluation can leverage SIMD (AVX2, NEON) via Arrow's compute module when compiled with appropriate features.

## Improvement Opportunities
- Investigate in-memory compression for intermediate results (e.g., dictionary re-encoding post-filter).
- Explore GPU device memory management integration to keep Arrow buffers resident on accelerators.
