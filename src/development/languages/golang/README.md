# Golang <img align="left" width="48" height="48" src="../../../data/images/logo_256x256.png">

### Quick links
- [.. up dir](../README.md)
* [Package Management](#package-management)
  * [Updating dependencies](#updating-dependencies)
* [Configuration](#Configuration)
  * [goformat](#goformat)
    * [indent spaces](#indent-spaces)
* [Relational databases](#relational-databases)
  * [Getting started](#getting-started)
    * [Freeing resources](#freeing-resources)
  * [Concurrancy](#concurrancy)
    * [Consuming connections](#consuming-connections)
    * [Transactions](#transactions)
    * [Deadlock with Transactions](#deadlock-with-transactions)

### Linked pages
* [Web](web/README.md)

# Build

## Build example
```bash
$ go build examples/
```

# Package Management

## Updating dependencies

### Update a specific dependency
* `-m` list modules instead of packages
* `-u` include available upgrade information

1. Display latest versions for a specific module
   ```bash
   $ go list -m -u github.com/phR0ze/n

   # use the -mod=mod for a vendored directory
   $ go list -mod=mod -m -u github.com/phR0ze/n
   github.com/phR0ze/n v1.2.9 [v1.3.0]
   ```

2. Upgrade the specific dependency
   ```bash
   $ go get -u github.com/phR0ze/n@v1.3.0
   ```

3. Clean up 
   ```bash
   $ go mod tidy
   ```

### Update all dependencies
```bash
$ go get -u
$ go mod tidy
$ make
```

# Configuration

## goformat
Typically these days most devs use VSCode and the `gopls` language server which automatically calls 
teh `gofmt` option to format your code. The only issue is that `gofmt` automatically uses tabs 
instead of spaces.

**References**
* [goformat by mbenkmann](https://github.com/mbenkmann/goformat)
* [gopls format setting](https://github.com/golang/vscode-go/blob/master/docs/settings.md#goformattool)

### indent spaces
Visual Studio Code has its own tab and space settings that I prefer to goformats. So we need to 
align the two. `go fmt` provides hooks for editors and version control systems that allow for 
overrides when autoformat it applied.

***-r option is a space delimited set of overrides***
```
goformat -r "indent=4"
```

# Relational databases
Golang provides a standard library `database/sql` package to access relational databases.

Resources:
* [oo.dev - Accessing relational databases](https://go.dev/doc/database/)
* [go.dev - Managing connections](https://go.dev/doc/database/manage-connections)

The `database/sql` package provides a high level uniform facade that must have a underlying driver to 
drive the actual operations. You can choose a SQL database driver from the
[recommended list](https://github.com/golang/go/wiki/SQLDrivers). The `database/sql` package 
simplifies database access by reducing the need for you to manage connections. Just open a database 
handle e.g. `var db *sql.DB` and access the db as needed; calling `Close` only when needed to free 
resources, such as those held by retrieved rows or prpared statements.

## Getting started
The [Go.dev tutorial](https://go.dev/doc/tutorial/database-access) walks through the basics

Declaring `var Db *sql.DB` global simplifies things for the example but in production you'd want to 
pass the variable to functions that need it or wrap it in a configuration struct.

### Freeing resources
Typically you close resources by deferring a call to a `Close` function so that resources are 
released before the enclosing function exists.
```go
rows, err := db.Query("SELECT * FROM album WHERE artist = ?", artist)
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

// Loop through returned rows.
```

## Concurrancy
Fro the vast majority of programs, you needn't adjust the `sql.DB` connection pool defaults.

The `sql.DB` database handle is safe for concurrent use by multiple goroutines (meaning the handle is 
what other languages might call `thread-safe`. Each `sql.DB` instance manages a pool of active 
connections to the underlying database, creating new ones as needed for parallelism in your Go 
program. When you make a `sql.DBQuery` or `Exec` call `sql.DB` retrieves the available connection 
from the pool or, if needed, creates one.

### Consuming connections
Failing to `Close` db results for `db.Query()` e.g. `defer rows.Close()` will leave the connection 
open and unavailable to other connection pool requests. `rows.Close()` is a idempotent i.e. harmless 
to call repeatedly. Don't `defer` within a loop

* [Golang DB SQL](https://medium.com/remotepanda-blog/golang-database-sql-chapter-9-540782555838)
* [Using Query for a statement that doesn't return rows](http://go-database-sql.org/surprises.html)

### Transactions
The `database/sql` package includes functions you can use when a database may assign implicit meaning 
to a sequence of operations executed on a particular connection. The most common example is 
transactions, which typically start with a `BEGIN` command and end with a `COMMIT` or `ROLLBACK` 
command and include all the commands issued on the connection between those commands in the overall 
transaction. For these cases `DB.Conn` obtains a dedicated connection a `sql.Conn`. The `sql.Conn` 
has methods `BeginTx, ExecContext, PintContext, PrepareContext, QueryContext` and `QueryRowContext` 
that behave like the equivalent methods on `DB` but only use the dedicated connection.

When finished with the dedicated connection, your code MUST release it using `conn.Close()`

* [Dedicated Connections](https://go.dev/doc/database/manage-connections#dedicated_connections)

### Deadlock with transactions
When using a transaction, take care not to call the non-transaction `sql.DB` methods directly, too, 
as those will execute outside the transaction, giving your code an inconsistent view of the state of 
the database or even causing `deadlocks` [see Best practices section](https://go.dev/doc/database/execute-transactions).

References:
* [Transactions spanning deadlock](https://github.com/golang/go/issues/3857)
* [Stack overflow](https://stackoverflow.com/questions/37769338/dead-lock-in-golang-database-sql)
* [Simutaneous Queries](https://go.googlesource.com/go/+/86743e7d8652c316b5f77a84ffc83244ee10a41b/src/database/sql/sql_test.go#1982)
* [Transactions deadlock](https://go.googlesource.com/go/+/86743e7d8652c316b5f77a84ffc83244ee10a41b/src/database/sql/sql_test.go#2836)

For example in the code below the `db.Query` call is not using the same session and if `select_statement`
references anything that was written by `insert_statement` it will block. Use `tx.Query` instead.
```go
tx, _ := db.Begin()
tx.Exec(insert_statement)
rows, _ := db.Query(select_statement)
rows.Close()
tx.Exec(insert_statement_2)
tx.Commit()
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
