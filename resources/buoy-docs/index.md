---
title: "PostgreSQL"
summary: "Use bound PostgreSQL parameters, exact values, streaming cursors, and structured database errors."
category: "Overview"
order: 0
keywords: ["postgresql", "database", "sql", "libpq", "parameters"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# PostgreSQL

pgsql implements Buoy's Database primitives through PgDatabase and supplies PostgreSQL-specific query, streaming, prepared statement, notification, COPY, and large-object APIs. Existing Database and RowSet consumers can retain their abstraction while using this backend.

## Values and execution

Parameters are bound out of band through libpq, never spliced into SQL. The placeholder scanner rewrites question marks to numbered parameters while respecting strings, quoted identifiers, nested comments, and dollar quotes. Native numbered parameters pass through; mixing both styles rejects.

Use ValueText for exact numeric values, ValueBytes for bytea, ValueJson for JSON, and ColumnTypeOid for server types. The generic DbValue numeric path still uses Double. QueryStream reserves its connection while holding only one chunk of result rows.

## Runtime and scope

The package bundles libpq 18 with OpenSSL native slices and license resources. Use a connection from one thread at a time; Cancel is the cross-thread exception. Inherited Async mirrors still run synchronously. There is no pipeline mode or binary result format.

## Read next

- [Overview](index.md)
- [Getting started](getting-started.md)
- [API reference](api.md)
