# Storage Integration

## Overview
DataFusion integrates with multiple storage formats and services, leveraging Apache Arrow for columnar transport
and the `object_store` abstraction for cloud-native data access.

## Supported File Formats
- **Parquet**: Primary columnar format with predicate and projection pushdown via metadata statistics.
- **CSV**: Flexible ingestion with schema inference, header detection, and delimiter customization.
- **JSON**: Line-delimited JSON reader for semi-structured data.
- **Avro**: Reader leveraging Apache Avro schema evolution support.
- **IPC/Arrow**: Direct consumption of Arrow file and stream formats.

## Columnar Optimizations
- Parquet reader leverages row group pruning, column chunk selection, and async prefetching.
- Uses Arrow's columnar arrays to minimize deserialization overhead.
- Supports dictionary re-use and encoding-aware conversions to reduce CPU cost.

## Object Store Abstraction
- `object_store::ObjectStore` trait wraps cloud storage (S3, GCS, Azure Blob) and local filesystems.
- Listing tables via `ListingTable` unify path-based datasets across stores with partition discovery.
- Credentials and retry policies configured through `ObjectStoreUrl` and runtime environment.

## Partitioning and Indexing
- Hive-style directory partitioning automatically mapped to schema columns.
- Parquet metadata caches store file-level statistics for partition pruning.
- Lacks built-in secondary indexing; relies on partitioning and data layout for pruning.

## Compression
- Parquet compression codecs (Snappy, Gzip, Zstd, Brotli) supported through `parquet` crate.
- CSV/JSON support gzip-compressed sources via async streams with decompression wrappers.

## Improvement Opportunities
- Add support for Delta Lake and Iceberg table metadata for ACID semantics.
- Introduce materialized view management with incremental refresh leveraging object store notifications.
