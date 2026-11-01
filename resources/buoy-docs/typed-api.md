---
title: "pgsql typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["PgDatabase", "PgException", "PgNotification", "PgParam", "PgRowSet", "PgStatement", "PgStreamRowSet", "PgType"]
dependencies: []
symbols: ["PgDatabase", "PgException", "PgNotification", "PgParam", "PgRowSet", "PgStatement", "PgStreamRowSet", "PgType"]
related: ["api.md", "getting-started.md", "index.md"]
---

# pgsql typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## PgType

`Public Class PgType`

Well-known PostgreSQL type OIDs, as reported by PgRowSet.ColumnTypeOid and accepted by PgParam.OfTyped.

### Constants

| Name | Type | Value / Default | Description |
| --- | --- | --- | --- |
| `Bool` | `Integer` | `16` | — |
| `Bytea` | `Integer` | `17` | — |
| `Int8` | `Integer` | `20` | — |
| `Int2` | `Integer` | `21` | — |
| `Int4` | `Integer` | `23` | — |
| `Text` | `Integer` | `25` | — |
| `Oid` | `Integer` | `26` | — |
| `Json` | `Integer` | `114` | — |
| `Float4` | `Integer` | `700` | — |
| `Float8` | `Integer` | `701` | — |
| `Varchar` | `Integer` | `1043` | — |
| `Date` | `Integer` | `1082` | — |
| `Time` | `Integer` | `1083` | — |
| `Timestamp` | `Integer` | `1114` | — |
| `TimestampTz` | `Integer` | `1184` | — |
| `Interval` | `Integer` | `1186` | — |
| `Numeric` | `Integer` | `1700` | — |
| `Uuid` | `Integer` | `2950` | — |
| `Jsonb` | `Integer` | `3802` | — |

## PgSql functions

### PgTranslatePlaceholders

`Public Function PgTranslatePlaceholders(sql As String) As String`

Rewrites each `?` placeholder to `$1`, `$2`, … and each `??` to a literal `?`, leaving string literals, quoted identifiers, comments and dollar-quoted bodies untouched. SQL already written with `$n` placeholders passes through; mixing the two styles throws.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

## PgException

`Public Class PgException Inherits DatabaseException`

A server or client error with PostgreSQL's structured fields. Inherits DatabaseException, so code written against the stdlib Database surface still catches it. SqlState is the five-character SQLSTATE code (e.g. "23505" unique_violation, "57014" query_canceled); it is "" for client-side failures such as a refused connection.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `SqlState` | `String` | No | No | `""` | — |
| `Severity` | `String` | No | No | `""` | — |
| `Detail` | `String` | No | No | `""` | — |
| `Hint` | `String` | No | No | `""` | — |
| `Position` | `Integer` | No | No | `0` | 1-based character position of the error in the statement, or 0. |
| `SchemaName` | `String` | No | No | `""` | — |
| `TableName` | `String` | No | No | `""` | — |
| `ColumnName` | `String` | No | No | `""` | — |
| `ConstraintName` | `String` | No | No | `""` | — |

### Constructor

`Public Sub Constructor(msg As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `msg` | `String` | `ByVal` | Required |

Returns: no value.

### IsConstraintViolation

`Public Function IsConstraintViolation() As Boolean`

True for SQLSTATE class 23 (integrity constraint violations).

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### IsRetryable

`Public Function IsRetryable() As Boolean`

True for serialization failures and deadlocks, which are safe to retry.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

## PgParam

`Public Class PgParam`

One bound parameter value, typed the way PostgreSQL types the equivalent literal so it resolves exactly as written-out SQL would: an Integer is int4 (int8 beyond that range, like `42` vs `9999999999`), a Double is numeric (like `2.5`; NaN and ±Infinity included), a Boolean is bool, and a String is untyped (like `'text'`), so the server infers dates, uuids, json and enums from context. `repeat('x', ?)`, `round(?, 2)`, `? IS NULL OR col = ?` and `? + ?` all work. A NULL has no type, so where nothing else supplies one (`? IS NULL` with a NULL) write `?::type`. Use OfTyped or a cast to choose a type yourself. Bytes travel as binary bytea.

### OfNull

`Public Shared Function OfNull() As PgParam`

No parameters.

#### Returns

| Type |
| --- |
| `PgParam` |

### OfString

`Public Shared Function OfString(v As String) As PgParam`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `v` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgParam` |

### OfInteger

`Public Shared Function OfInteger(v As Integer) As PgParam`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `v` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgParam` |

### OfDouble

`Public Shared Function OfDouble(v As Double) As PgParam`

Sent as numeric with full round-trip precision, so it converts to a float8 column exactly; NaN and ±Infinity included.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `v` | `Double` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgParam` |

### OfBoolean

`Public Shared Function OfBoolean(v As Boolean) As PgParam`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `v` | `Boolean` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgParam` |

### OfBytes

`Public Shared Function OfBytes(v As Bytes) As PgParam`

Binary bytea. The bytes are copied, so later changes to `v` don't leak in.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `v` | `Bytes` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgParam` |

### OfTyped

`Public Shared Function OfTyped(valueText As String, typeOid As Integer) As PgParam`

Text in PostgreSQL's input syntax for the type `typeOid` (see PgType).

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `valueText` | `String` | `ByVal` | Required |
| `typeOid` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgParam` |

### FromDbValue

`Public Shared Function FromDbValue(v As DbValue) As PgParam`

Converts a stdlib DbValue, keeping its kind.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `v` | `DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgParam` |

### FromDbValues

`Public Shared Function FromDbValues(values As Array Of DbValue) As Array Of PgParam`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `values` | `Array Of DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Array Of PgParam` |

## PgRowSet

`Public Class PgRowSet Inherits RowSet`

PostgreSQL RowSet. A plain Query buffers the whole result client-side; a cursor from PgDatabase.QueryStream holds only one chunk at a time. Beyond the stdlib RowSet surface it reads values exactly (ValueText), as bytea (ValueBytes) or JSON (ValueJson), and reports column types and the affected-row count.

### HasMoreRows

`Public Overrides Function HasMoreRows() As Boolean`

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### NextRow

`Public Overrides Sub NextRow()`

No parameters.

Returns: no value.

### ColumnCount

`Public Overrides Function ColumnCount() As Integer`

No parameters.

#### Returns

| Type |
| --- |
| `Integer` |

### ColumnName

`Public Overrides Function ColumnName(index As Integer) As String`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `index` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### ColumnIndex

`Public Function ColumnIndex(column As String) As Integer`

The 0-based index of the column named `column`, or -1. An exact match wins; otherwise the first case-insensitive match is used.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `column` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### ColumnTypeOid

`Public Function ColumnTypeOid(index As Integer) As Integer`

The column's PostgreSQL type OID (compare with the PgType constants).

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `index` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### Value

`Public Overrides Function Value(column As String) As DbValue`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `column` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `DbValue` |

### Value (overload 2)

`Public Overrides Function Value(index As Integer) As DbValue`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `index` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `DbValue` |

### IsNull

`Public Function IsNull(index As Integer) As Boolean`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `index` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Boolean` |

### IsNull (overload 2)

`Public Function IsNull(column As String) As Boolean`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `column` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Boolean` |

### ValueText

`Public Function ValueText(index As Integer) As String`

The server's exact text for the value ("" for NULL): full-precision numeric, timestamps, uuids, intervals, arrays, and so on.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `index` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### ValueText (overload 2)

`Public Function ValueText(column As String) As String`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `column` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### ValueBytes

`Public Function ValueBytes(index As Integer) As Bytes`

A bytea column decoded to its raw bytes; any other type yields the UTF-8 bytes of its text. NULL yields an empty buffer.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `index` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Bytes` |

### ValueBytes (overload 2)

`Public Function ValueBytes(column As String) As Bytes`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `column` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Bytes` |

### ValueJson

`Public Function ValueJson(index As Integer) As JSONItem`

A json/jsonb (or any JSON-text) column parsed into a JSONItem; NULL yields an empty null item.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `index` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `JSONItem` |

### ValueJson (overload 2)

`Public Function ValueJson(column As String) As JSONItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `column` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `JSONItem` |

### RowCount

`Public Function RowCount() As Integer`

Rows in the buffered result; for a streaming cursor, rows in the current chunk only.

No parameters.

#### Returns

| Type |
| --- |
| `Integer` |

### AffectedRows

`Public Function AffectedRows() As Integer`

Rows inserted, updated, deleted, copied or selected by the statement. For a streaming cursor this is known only once it is exhausted.

No parameters.

#### Returns

| Type |
| --- |
| `Integer` |

### CommandTag

`Public Function CommandTag() As String`

The command tag, e.g. "INSERT 0 3" or "UPDATE 2".

No parameters.

#### Returns

| Type |
| --- |
| `String` |

### Close

`Public Overrides Sub Close()`

No parameters.

Returns: no value.

### Destructor

`Public Sub Destructor()`

Frees the result when a discarded RowSet is released.

No parameters.

Returns: no value.

## PgStreamRowSet

`Public Class PgStreamRowSet Inherits PgRowSet`

A streaming cursor over one statement, fetched from the server in chunks (PgDatabase.QueryStream). The connection is dedicated to the cursor until it is exhausted or closed: other calls on the PgDatabase throw meanwhile. Close reads and discards any remaining rows; CancelAndClose asks the server to stop the statement instead (which aborts an open transaction). A cursor left open across Disconnect/Reconnect simply ends: it never touches the new connection.

### HasMoreRows

`Public Overrides Function HasMoreRows() As Boolean`

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### CancelAndClose

`Public Sub CancelAndClose()`

Asks the server to cancel the statement, then closes. Cheaper than Close for a huge result you no longer need; inside a transaction the cancellation aborts the transaction.

No parameters.

Returns: no value.

### Close

`Public Overrides Sub Close()`

No parameters.

Returns: no value.

### Destructor

`Public Sub Destructor()`

No parameters.

Returns: no value.

## PgStatement

`Public Class PgStatement`

A server-side prepared statement (PgDatabase.Prepare): parsed and planned once, executed many times. Close deallocates it; a statement is also closed automatically when released. Every `?` is a placeholder here (the parameters come later); write `??` for the jsonb `?` operator.

### Name

`Public Function Name() As String`

The server-side statement name.

No parameters.

#### Returns

| Type |
| --- |
| `String` |

### Query

`Public Function Query(params As Array Of PgParam) As PgRowSet`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `params` | `Array Of PgParam` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgRowSet` |

### Query (overload 2)

`Public Function Query(params As Array Of DbValue) As PgRowSet`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `params` | `Array Of DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgRowSet` |

### Query (overload 3)

`Public Function Query() As PgRowSet`

No parameters.

#### Returns

| Type |
| --- |
| `PgRowSet` |

### Execute

`Public Function Execute(params As Array Of PgParam) As Integer`

Runs the statement and returns the affected-row count.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `params` | `Array Of PgParam` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### Execute (overload 2)

`Public Function Execute(params As Array Of DbValue) As Integer`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `params` | `Array Of DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### QueryStream

`Public Function QueryStream(params As Array Of PgParam, chunkSize As Integer = 1000) As PgStreamRowSet`

Streams the statement's rows in chunks (see PgDatabase.QueryStream).

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `params` | `Array Of PgParam` | `ByVal` | Required |
| `chunkSize` | `Integer` | `ByVal` | `1000` |

#### Returns

| Type |
| --- |
| `PgStreamRowSet` |

### Close

`Public Sub Close()`

Deallocates the statement. While a streaming cursor holds the connection, the deallocation is queued and runs on the next call.

No parameters.

Returns: no value.

### Destructor

`Public Sub Destructor()`

No parameters.

Returns: no value.

## PgNotification

`Public Class PgNotification`

One NOTIFY delivered to a session that LISTENs on its channel.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Channel` | `String` | No | No | `""` | — |
| `Payload` | `String` | No | No | `""` | — |
| `ProcessId` | `Integer` | No | No | `0` | The backend process id of the notifying session. |

## PgDatabase

`Public Class PgDatabase Inherits Database`

PostgreSQL implementation of the stdlib Database. Construct, then Connect with a libpq connection string or URI, e.g. "host=localhost dbname=app user=app sslmode=require" or "postgresql://app@localhost/app?sslmode=verify-full".  A PgDatabase is one connection: use it from one thread at a time. The exception is Cancel, which may be called from any thread.

### Constructor

`Public Sub Constructor()`

No parameters.

Returns: no value.

### Connect

`Public Overrides Sub Connect(connectionString As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `connectionString` | `String` | `ByVal` | Required |

Returns: no value.

### Disconnect

`Public Overrides Sub Disconnect()`

No parameters.

Returns: no value.

### IsConnected

`Public Overrides Function IsConnected() As Boolean`

True while the connection is open and healthy (False once the server has gone away, even before Disconnect).

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### Reconnect

`Public Sub Reconnect()`

Re-establishes a broken connection with the original parameters. Session state (prepared statements, LISTENs, SET values) is lost, and an open streaming cursor ends.

No parameters.

Returns: no value.

### Cancel

`Public Function Cancel() As Boolean`

Asks the server to cancel the statement currently running on this connection. Safe to call from another thread. Returns True when the request was delivered; the running call then fails with SQLSTATE 57014.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### Destructor

`Public Sub Destructor()`

No parameters.

Returns: no value.

### ServerVersion

`Public Function ServerVersion() As Integer`

The server version as an integer, e.g. 180006 for 18.6.

No parameters.

#### Returns

| Type |
| --- |
| `Integer` |

### BackendPid

`Public Function BackendPid() As Integer`

The process id of the backend serving this connection.

No parameters.

#### Returns

| Type |
| --- |
| `Integer` |

### ParameterStatus

`Public Function ParameterStatus(name As String) As String`

A server-reported setting such as "server_version", "TimeZone" or "standard_conforming_strings"; "" if unknown.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `name` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### InTransaction

`Public Function InTransaction() As Boolean`

True inside a transaction block, including a failed one awaiting ROLLBACK.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### InFailedTransaction

`Public Function InFailedTransaction() As Boolean`

True inside a transaction that has failed and must be rolled back.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### QuoteIdentifier

`Public Function QuoteIdentifier(name As String) As String`

Quotes `name` as an SQL identifier ("my table" → "\"my table\"").

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `name` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### QuoteLiteral

`Public Function QuoteLiteral(value As String) As String`

Quotes `value` as an SQL string literal. Prefer bound parameters; this is for the few places they're not allowed (some DDL and utility statements).

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `value` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### Query

`Public Overrides Function Query(sql As String) As RowSet`

Runs `sql` with no parameters. Several `;`-separated statements are allowed; the result of the last one is returned.  Placeholder rule for every query method: with parameters, each `?` is a placeholder; with none, a lone `?` is left alone (it can only be the jsonb operator); `??` always means a literal `?`.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `RowSet` |

### Query (overload 2)

`Public Overrides Function Query(sql As String, params As Array Of DbValue) As RowSet`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `RowSet` |

### Query (overload 3)

`Public Function Query(sql As String, params As Array Of PgParam) As RowSet`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of PgParam` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `RowSet` |

### QueryPg

`Public Function QueryPg(sql As String) As PgRowSet`

Query returning the PostgreSQL row set type directly, so its extra accessors (ValueText, ValueBytes, AffectedRows …) need no cast.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgRowSet` |

### QueryPg (overload 2)

`Public Function QueryPg(sql As String, params As Array Of PgParam) As PgRowSet`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of PgParam` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgRowSet` |

### QueryPg (overload 3)

`Public Function QueryPg(sql As String, params As Array Of DbValue) As PgRowSet`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgRowSet` |

### Execute

`Public Function Execute(sql As String) As Integer`

Runs a statement and returns its affected-row count.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### Execute (overload 2)

`Public Function Execute(sql As String, params As Array Of PgParam) As Integer`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of PgParam` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### Execute (overload 3)

`Public Function Execute(sql As String, params As Array Of DbValue) As Integer`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### QueryScalar

`Public Function QueryScalar(sql As String, params As Array Of PgParam) As DbValue`

The first column of the first row (NULL when there are no rows).

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of PgParam` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `DbValue` |

### QueryScalar (overload 2)

`Public Function QueryScalar(sql As String, params As Array Of DbValue) As DbValue`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `DbValue` |

### QueryScalar (overload 3)

`Public Function QueryScalar(sql As String) As DbValue`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `DbValue` |

### QueryStream

`Public Function QueryStream(sql As String, params As Array Of PgParam, chunkSize As Integer = 1000) As PgStreamRowSet`

Streams one statement's rows from the server in chunks of `chunkSize`, so memory stays flat however large the result is. The connection is reserved for the cursor until it is exhausted or closed.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of PgParam` | `ByVal` | Required |
| `chunkSize` | `Integer` | `ByVal` | `1000` |

#### Returns

| Type |
| --- |
| `PgStreamRowSet` |

### QueryStream (overload 2)

`Public Function QueryStream(sql As String, params As Array Of DbValue, chunkSize As Integer = 1000) As PgStreamRowSet`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `params` | `Array Of DbValue` | `ByVal` | Required |
| `chunkSize` | `Integer` | `ByVal` | `1000` |

#### Returns

| Type |
| --- |
| `PgStreamRowSet` |

### QueryStream (overload 3)

`Public Function QueryStream(sql As String) As PgStreamRowSet`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgStreamRowSet` |

### Prepare

`Public Function Prepare(sql As String) As PgStatement`

Parses and plans `sql` once on the server. Parameter types are inferred from the statement; add `?::type` casts where the context is ambiguous.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgStatement` |

### LastInsertId

`Public Overrides Function LastInsertId() As Integer`

The id generated by the most recent INSERT through this connection. For a single `INSERT INTO <table>` (plain or prepared, comments allowed) it reads that table's own serial/identity sequence with currval(), so a trigger that touches another sequence can't skew it, and it returns 0 for a table without one. After WITH, COPY, MERGE or a multi-statement string it falls back to lastval(). Returns 0 when there is no id. Prefer `INSERT … RETURNING id` in new code.

No parameters.

#### Returns

| Type |
| --- |
| `Integer` |

### Listen

`Public Sub Listen(channel As String)`

Subscribes this session to `channel`. Notifications arrive between transactions; read them with WaitForNotification or PendingNotifications.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `channel` | `String` | `ByVal` | Required |

Returns: no value.

### Unlisten

`Public Sub Unlisten(channel As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `channel` | `String` | `ByVal` | Required |

Returns: no value.

### UnlistenAll

`Public Sub UnlistenAll()`

No parameters.

Returns: no value.

### Notify

`Public Sub Notify(channel As String, payload As String = "")`

Sends a notification. Inside a transaction it is delivered on COMMIT.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `channel` | `String` | `ByVal` | Required |
| `payload` | `String` | `ByVal` | `""` |

Returns: no value.

### PendingNotifications

`Public Function PendingNotifications() As Array Of PgNotification`

Every notification already received, without blocking.

No parameters.

#### Returns

| Type |
| --- |
| `Array Of PgNotification` |

### WaitForNotification

`Public Function WaitForNotification(timeoutMs As Integer) As PgNotification`

Waits up to `timeoutMs` milliseconds (negative: forever) for the next notification; returns Nil on timeout.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `timeoutMs` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `PgNotification` |

### CopyFrom

`Public Function CopyFrom(sql As String, data As String) As Integer`

Runs a `COPY … FROM STDIN` statement, sending `data` (already in the statement's format: text, csv, …). Returns the number of rows copied.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `data` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### CopyFrom (overload 2)

`Public Function CopyFrom(sql As String, rows As Array Of Array Of DbValue) As Integer`

COPY FROM STDIN in text format, one row per entry (see CopyRow).

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |
| `rows` | `Array Of Array Of DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### CopyTo

`Public Function CopyTo(sql As String) As String`

Runs a `COPY … TO STDOUT` statement and returns everything it wrote.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sql` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### CopyRow

`Public Shared Function CopyRow(fields As Array Of DbValue) As String`

Encodes one row for COPY's text format: tab-separated, `\N` for NULL, backslash escapes for tab/newline/CR/backslash, newline-terminated.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `fields` | `Array Of DbValue` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### CreateLargeObject

`Public Function CreateLargeObject(data As Bytes) As Integer`

Stores `data` as a new large object and returns its OID.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `data` | `Bytes` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### WriteLargeObject

`Public Sub WriteLargeObject(oid As Integer, data As Bytes)`

Replaces the contents of large object `oid` with `data`.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `oid` | `Integer` | `ByVal` | Required |
| `data` | `Bytes` | `ByVal` | Required |

Returns: no value.

### ReadLargeObject

`Public Function ReadLargeObject(oid As Integer) As Bytes`

Reads the whole of large object `oid`.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `oid` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Bytes` |

### DeleteLargeObject

`Public Sub DeleteLargeObject(oid As Integer)`

Deletes large object `oid`.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `oid` | `Integer` | `ByVal` | Required |

Returns: no value.
