---
title: "PostgreSQL getting started"
summary: "Install and use PostgreSQL with explicit configuration and resource ownership."
category: "Guides"
order: 10
keywords: ["postgresql", "database", "sql", "libpq", "parameters"]
related: ["index.md", "api.md", "typed-api.md"]
---

# PostgreSQL getting started

Copy the pgsql archive into packages, including its matching runtime payload. Configure a connection using libpq key=value text or a URI from your application settings; do not embed database passwords in source. The example performs a parameterized SELECT against a local app database and closes the row set and connection explicitly.

```buoy
Import pgsql

Sub Main()
    Dim database As New PgDatabase()
    Try
        database.Connect("host=localhost dbname=app user=app sslmode=require")
        Dim rows As PgRowSet = database.QueryPg("SELECT ?::text AS message", [PgParam.OfString("Hello")])
        Try
            For Each row As RowSet In rows
                Print rows.ValueText("message")
            Next
        Finally
            rows.Close()
        End Try
    Finally
        database.Disconnect()
    End Try
End Sub
```

## Resource lifetime

Close buffered row sets when finished. A streaming cursor owns the connection until exhausted or closed; other operations reject while it is active. Close discards remaining rows; CancelAndClose asks the server to stop instead. Reconnect loses session state and invalidates statements prepared on the prior connection.

## Binding and precision

PgParam.OfString lets PostgreSQL infer context. OfInteger selects int4 or int8 by range, OfDouble sends full-precision numeric text, OfBoolean uses bool, and OfBytes sends binary bytea. A NULL has no inferred type: cast a placeholder when the surrounding SQL supplies no type.

Use QuoteIdentifier/QuoteLiteral only where SQL grammar prevents ordinary value binding. For COPY FROM STDIN or TO STDOUT, use CopyFrom/CopyTo rather than a general Query/Execute call. Large objects require an appropriate transaction.

## Read next

- [Overview](index.md)
- [Getting started](getting-started.md)
- [API reference](api.md)
