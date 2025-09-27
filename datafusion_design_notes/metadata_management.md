# Metadata Management

## Overview
Metadata in DataFusion is centered on the `CatalogProvider`, `SchemaProvider`, and `TableProvider` traits,
which abstract storage backends and expose schema information to planners and execution components.

## Table and Schema Storage
- **CatalogProvider** manages named catalogs (default `datafusion`). Implementations can back onto in-memory maps,
  Hive Metastore, Glue, or custom registries.
- **SchemaProvider** stores collections of tables within a catalog. The default implementation is memory-backed.
- **TableProvider** exposes logical schema, statistics, and execution plans for a table. Concrete implementations include
  `ListingTable` (object storage files), `MemoryTable`, `ParquetExec`, and external connectors.

## Name Resolution
- The logical planner uses `DFSchema` and `DFField` to manage column qualification, including aliases and wildcard expansion.
- Resolves identifiers via `SessionState`'s catalog list, falling back to default catalog/schema when unspecified.
- Supports temporary views and CTEs stored in session-specific state maps.

## Caching Strategies
- **TableProvider::statistics** enables caching file-level metadata (e.g., row count, byte size) for pruning.
- `ListingTableCache` caches file listings per table to avoid repeated object store scans.
- Session-level caches store prepared statements and logical plan representations for repeated queries.

## External Integrations
- Hive-style partition discovery supported via directory structure parsing in `ListingTableUrl`. Partition columns
  inferred and appended to schema metadata.
- AWS/GCP/Azure object stores supported through `object_store` crate implementations, enabling credential injection
  and secure metadata access.
- Integration with DataFusion's catalog service in Ballista enables cluster-wide metadata synchronization.

## Improvement Opportunities
- Introduce unified metadata snapshotting to coordinate statistics and schema versions across distributed executors.
- Provide incremental refresh strategies (e.g., event-based updates) to reduce latency in catalog changes.
