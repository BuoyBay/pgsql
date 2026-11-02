# PostgreSQL for Buoy · 0.1.0

A PostgreSQL backend for the stdlib `Database` base class. It replaces the
toolchain's bundled `postgres` package where that package falls short. Install
`packages/pgsql-0.1.0.buipkg` in another project's `packages/` directory and
use `Import pgsql`. Build it with `buoy-run tools/build_pgsql.bui`.

`PgDatabase` implements the six `Database` primitives, so `Query Of T`,
`Insert`/`Update` for `@DatabaseRecord` classes, transactions and the `*Async`
mirrors all work unchanged. Code written against `Database`, `RowSet` and
`DatabaseException` moves over by changing the constructor.

```buoy
Import pgsql

Sub Main()
    Dim db As New PgDatabase()
    db.Connect("host=localhost dbname=app user=app sslmode=require")
    Dim rs As PgRowSet = db.QueryPg("SELECT id, total FROM orders WHERE customer = ? AND total > ?", _
        [PgParam.OfString(name), PgParam.OfDouble(100.0)])
    For Each row As RowSet In rs
        Print rs.ValueText("total")          ' exact numeric text
    Next
    db.Disconnect()
End Sub
```

## What it fixes compared with the bundled `postgres` package

| Bundled `postgres` 18.4.0 | `pgsql` |
| --- | --- |
| `?` values are escaped and spliced into the SQL text, then sent with `PQexec` | Values are bound out of band with `PQexecParams` and never become SQL text. `?` is rewritten to `$n`, skipping literals, `E''` strings, quoted identifiers, nested comments and dollar quotes. Native `$1` SQL passes through unchanged; mixing `$n` and `?` in one statement throws |
| Doubles are sent with `Str()`, which keeps 6 significant digits (`123456789.123` becomes `1.23457e+08`) | Doubles are sent with the shortest text that parses back to the same value, including `NaN` and `±Infinity` |
| Every result set is fully buffered | `QueryStream` uses libpq's chunked-rows mode, so only one chunk is held in memory at a time |
| No prepared statements | `Prepare` returns a `PgStatement` (`Query`, `Execute`, `QueryStream`, `Close`). A statement closed while a cursor holds the connection is deallocated on the next call |
| No LISTEN/NOTIFY | `Listen`, `Unlisten`, `Notify`, `PendingNotifications`, `WaitForNotification(timeoutMs)` |
| No COPY | `CopyFrom` (raw text or CSV, or rows of `DbValue`), `CopyTo`, `PgDatabase.CopyRow`. Each call takes exactly one COPY statement |
| No large objects | `CreateLargeObject`, `ReadLargeObject`, `WriteLargeObject`, `DeleteLargeObject` |
| No way to stop a running query | `Cancel()` works from any thread; the running call fails with SQLSTATE `57014` |
| `DatabaseException` carries only a message | `PgException` (a `DatabaseException`) adds `SqlState`, `Detail`, `Hint`, `Position`, `SchemaName`, `TableName`, `ColumnName`, `ConstraintName`, `Severity`, plus `IsConstraintViolation()` and `IsRetryable()` |
| bytea, numeric, uuid, json and timestamps are strings or rounded Doubles | `ValueText` gives the server's exact text, `ValueBytes` decodes bytea, `ValueJson` parses json/jsonb, `ColumnTypeOid` reports the type, and `PgParam.OfBytes` sends binary bytea |
| `LastInsertId` uses `lastval()`, which goes wrong if a trigger touches another sequence | For a single `INSERT INTO <table>`, plain or prepared and with or without leading comments, it reads that table's own serial/identity sequence. A table without one returns 0. After `WITH`, `COPY`, `MERGE`, `DO`, `CALL`, `EXECUTE` or multi-statement SQL it falls back to `lastval()`. `SET`, `RESET` and DDL refresh its cached sequence names, and quoted names are matched case-sensitively. A SELECT doesn't change it, so an insert made by a function called from a SELECT isn't followed. Inside a transaction the lookup runs under a savepoint, so it can't abort the caller's transaction |
| No affected-row count | `Execute` returns it; also `AffectedRows()` and `CommandTag()` on the row set |

## API

| API | Behavior |
| --- | --- |
| `Connect(conninfo)` / `Disconnect()` / `IsConnected()` | Takes a libpq key=value string or URI. Sets `client_encoding` to UTF8. `IsConnected` turns False if the server drops the connection |
| `Reconnect()` | Calls `PQreset`. Session state is lost, and statements prepared on the old connection are rejected rather than run against the new one |
| `Query(sql)` / `QueryPg(sql)` | Several `;`-separated statements are allowed; returns the last result |
| `Query`/`QueryPg`/`Execute`/`QueryScalar(sql, params)` | Params are `Array Of PgParam` or `Array Of DbValue`. `QueryPg` returns `PgRowSet`; `Execute` returns the affected-row count |
| `QueryStream(sql, params, chunkSize = 1000)` | Returns a `PgStreamRowSet`. The connection is reserved until the cursor is exhausted or closed, and other calls throw meanwhile. `Close` discards the remaining rows; `CancelAndClose` asks the server to stop instead. A cursor left open across `Disconnect`/`Reconnect` just ends; it never touches the new connection |
| `PgParam.OfString/OfInteger/OfDouble/OfBoolean/OfNull/OfBytes/OfTyped(text, oid)/FromDbValue` | Each value is typed the way PostgreSQL types the equivalent literal, so a parameter resolves exactly like written-out SQL. An Integer is int4 (int8 beyond int4 range), a Double is numeric at full precision, a Boolean is bool, and a String is untyped, so the server infers dates, uuids, json and enums from context. `repeat('x', ?)`, `round(?, 2)`, `? IS NULL OR col = ?` and `? + ?` all work. A NULL has no type, so write `?::type` where nothing else supplies one. Bytes are sent as binary bytea |
| Placeholders | With parameters, each `?` is a placeholder. With no parameters, a lone `?` is left alone (the jsonb operator). `??` always means a literal `?`. `Prepare` always treats `?` as a placeholder, since its parameters come later |
| COPY via `Query`/`Execute`/`QueryStream` | A `COPY … FROM STDIN` or `TO STDOUT` is refused before anything is sent, even inside multi-statement SQL, with an error pointing at `CopyFrom`/`CopyTo`. The statements after it don't run, and an open transaction isn't aborted. Server-side `COPY … TO/FROM 'file'` or `PROGRAM` still runs through `Execute` |
| `PgRowSet` | The `RowSet` methods plus `ValueText`, `ValueBytes`, `ValueJson`, `IsNull`, `ColumnIndex`, `ColumnTypeOid`, `RowCount`, `AffectedRows`, `CommandTag` |
| `QuoteIdentifier`, `QuoteLiteral` | For the few places binding isn't allowed (DDL, utility statements) |
| `ServerVersion`, `BackendPid`, `ParameterStatus(name)`, `InTransaction`, `InFailedTransaction` | Session inspection |
| `PgType.*` | Well-known type OIDs (`Int8`, `Numeric`, `Bytea`, `Jsonb`, `Uuid`, `TimestampTz`, …) |
| `PgTranslatePlaceholders(sql)` | The `?` → `$n` rewrite on its own, for inspection |

## Notes and limits

- A `PgDatabase` is one connection: use it from one thread at a time. `Cancel`
  is the only method meant to be called from another thread.
- In the stdlib `DbValue` surface, `numeric` still maps to `Double` because
  `DbValue` has no decimal kind, so `@DatabaseRecord` `Double` properties
  work. Use `ValueText` when every digit matters.
- Notifications are read by polling (`PendingNotifications`) or by blocking
  (`WaitForNotification`). There is no background listener thread or event.
  On macOS and Linux the wait uses `select()`, which only handles sockets
  below fd 1024; above that it falls back to polling every 10 ms.
- Server NOTICE messages go to stderr through libpq's default handler. Use
  `SET client_min_messages = warning` to silence them.
- The `*Async` mirrors are inherited from the stdlib and still run
  synchronously. Streaming and `Cancel` are the ways to avoid blocking on
  large or slow queries.
- No pipeline mode and no binary result format. Results are always text and
  decoded per type.
- The package bundles the same libpq 18.4 binaries (with OpenSSL) for all six
  targets as the toolchain's `postgres` package. Their licenses ship under
  `resources/pgsql-licenses/` and are copied next to built executables. Both
  packages can be installed side by side because the class names differ.
