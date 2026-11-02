# PostgreSQL Database / RowSet

A PostgreSQL backend for the stdlib `Database` base class. It replaces the
toolchain's bundled `postgres` package where that package falls short. Install
`packages/pgsql-0.1.0.buipkg` in another project's `packages/` directory and
use `Import pgsql`. Build it with `./dist/ahoy tools/build_pgsql.bui`.

`PgDatabase` implements the six `Database` primitives, so `Query Of T`,
`Insert`/`Update` for `@DatabaseRecord` classes, transactions and the `*Async`
mirrors all work unchanged. Code written against `Database`, `RowSet` and
`DatabaseException` moves over by changing the constructor.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add pgsql --git https://github.com/BuoyBay/pgsql.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## Example

```basic
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

## Build and API reference

Source is in `modules/`, with native/runtime inputs where required. Use the
[package guide](docs/API.md) for backend-specific build prerequisites, ownership
rules and supported targets. Rebuilding a native backend requires that target's
compiler and SDK; sealing an archive alone does not rebuild the backend.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```

## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
