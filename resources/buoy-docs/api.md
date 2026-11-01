---
title: "PostgreSQL API"
summary: "Public types, operations, error behavior, and limits for PostgreSQL."
category: "Reference"
order: 20
keywords: ["postgresql", "database", "sql", "libpq", "parameters"]
symbols: ["PgDatabase", "PgRowSet", "PgStreamRowSet", "PgParam", "PgStatement", "PgException", "PgType", "PgNotification", "PgTranslatePlaceholders", "PgDatabase.QueryPg", "PgDatabase.QueryStream"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# PostgreSQL API

| API | Contract |
| --- | --- |
| PgDatabase.Connect, Disconnect, IsConnected, Reconnect | Own one libpq connection; reconnect resets session state. |
| Query, QueryPg, Execute, QueryScalar | SQL with PgParam or DbValue arrays; QueryPg returns PgRowSet, Execute returns affected rows. |
| QueryStream | PgStreamRowSet with default chunk size 1000 and exclusive connection ownership. |
| Prepare | PgStatement with Query, Execute, QueryStream, and Close. |
| PgParam.OfString, OfInteger, OfDouble, OfBoolean, OfNull, OfBytes, OfTyped, FromDbValue | Typed bound values. |
| PgRowSet | Standard RowSet plus ValueText, ValueBytes, ValueJson, IsNull, ColumnIndex, ColumnTypeOid, RowCount, AffectedRows, CommandTag. |
| PgStreamRowSet.CancelAndClose | Cancel an unfinished streaming operation and release the connection. |
| Listen, Unlisten, UnlistenAll, Notify, PendingNotifications, WaitForNotification | Explicit notification registration and polling/blocking consumption. |
| CopyFrom, CopyTo, PgDatabase.CopyRow | Dedicated COPY streaming-protocol entry points; CopyTo returns buffered String data. |
| CreateLargeObject, ReadLargeObject, WriteLargeObject, DeleteLargeObject | Transaction-scoped large-object operations. |
| Cancel | Request cancellation from another thread; running query reports SQLSTATE 57014. |
| PgType, PgTranslatePlaceholders | PostgreSQL type OIDs and standalone placeholder inspection. |

## Errors and transactions

PgException inherits DatabaseException and exposes SqlState, Severity, Detail, Hint, Position, SchemaName, TableName, ColumnName, and ConstraintName. IsConstraintViolation and IsRetryable classify failures; callers own transaction rollback/retry policy.

ServerVersion, BackendPid, ParameterStatus, InTransaction, and InFailedTransaction inspect session state. LastInsertId attempts the inserted table's own sequence for straightforward INSERT statements and falls back to lastval for more complex statement forms. Prefer explicit RETURNING when the statement should return an identity directly.

## Read next

- [Overview](index.md)
- [Getting started](getting-started.md)
- [API reference](api.md)


[Full typed API: properties, parameters, defaults, and returns](typed-api.md)
