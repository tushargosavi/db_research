# SQL Frontend

## Overview
DataFusion exposes a SQL interface that translates SQL-92 compatible statements into logical plans using the
`sqlparser-rs` library and the `datafusion-sql` crate. The frontend is extensible, enabling custom functions and
statements while keeping close parity with Arrow DataFrame semantics.

## Parser Implementation
- Uses **`sqlparser-rs`** for lexical analysis and AST generation. DataFusion extends the parser with custom dialects
  for features like `SHOW TABLES`, `DESCRIBE`, and `CREATE EXTERNAL TABLE`.
- The SQL planner converts `sqlparser::ast::Statement` nodes into DataFusion logical plan nodes through the
  `SqlToRel` struct, mapping expressions recursively.

## Extensions and Custom Syntax
- Supports statements for data source registration (`CREATE EXTERNAL TABLE`, `CREATE VIEW`).
- Provides session-specific commands like `SET VARIABLE` and `CREATE FUNCTION` for scalar UDF registration.
- Recognizes DataFusion-specific COPY statements for CSV and Parquet ingestion in CLI contexts.

## Compliance and Feature Set
- Targets **SQL-92** compatibility with incremental support for newer constructs (e.g., `UNNEST`, `LATERAL VIEW`).
- Supports common DDL/DML: `SELECT`, `INSERT`, `UPDATE` (limited), `DELETE` (when table providers expose delete handles).
- Offers scalar, aggregate, and window functions aligned with ANSI SQL naming where possible.

## Built-in Functions
- Scalar functions implemented under `datafusion-expr/src/expr_fn.rs` and `datafusion-functions` modules include
  arithmetic, string, datetime, math, and conditional operators.
- Aggregate functions like `SUM`, `AVG`, `APPROX_DISTINCT`, `GROUPING` defined through `AggregateUDF` interfaces.
- Window functions (e.g., `ROW_NUMBER`, `RANK`, `LEAD/LAG`) reuse aggregate kernels with partition/order metadata.

## AST Structure
- `sqlparser-rs` AST includes enums such as `Statement`, `Expr`, `Query`, and `SelectItem`.
- DataFusion introduces wrapper enums (`DFSchema`, `DFField`) to bridge SQL identifiers with logical plan schemas.
- Visitor pattern implemented through `SqlToRel::sql_to_relation` and helper functions to traverse expressions,
  performing type coercion and name resolution during translation.

## Improvement Ideas
- Investigate parser-level hints or annotations (e.g., optimizer hints) to guide rule selection.
- Expand dialect support for ANSI SQL 2016 analytics, particularly advanced window framing and JSON features.
