# 4. Database Driven Responses

## designing a database model / data access layer / service layer

- Encapsulate the code for working with MySQL in a separate package.
- Data access layer, where each query is exposed as a purpose built method: GetUserByID(id string) (User, error) for example.

- the `internal` directory is used to hold non-application-specific code which could potentially be reused

### Additional Information
.
.
.
Understand this


Executing queries:

Go provides three different methods for executing databaes queries

DB.Query()
DB.QueryRow()
DB.Exec()

- In the SQL statements, we've used placeholder parameters `?` for passing data we want to insert
- Reason for using this is to avoid SQL injection attacks / we don't want to use string interpolation here


DB.Exec() steps:

i. Creates _prepared statement_, db parses and compiles that statement and prepares it for execution
ii. `Exec()` passes the parameter values to the database, db executes the prepared statement using these values. Values are passed after the statement compilation (db treats those values are pure data, statement injection can't occur because of this)
iv. Closes the statement on the DB

**Different Syntax for Placeholders**
MySQL and SQLite use the ? notation, but PostgreSQL uses the $N notation

`_, err := m.DB.Exec("INSERT INTO ... VALUES ($1, $2, $3)", ...)`

The database driver automatically converts the raw output from database to require native Go types


It is safer and better practice to use errors.Is() to compare the errors


- database/sql package essentially provides a standard interface between your Go application and the world of SQL databases
- as long as we use the `database/sql` package, the Go code we write, will generally be portable to other SQL databases

- Go doesn't manage `NULL` values in databases
- Go can't convert NULL into a string, so `rows.Scan` would throw an error

We should avoid NULL


### Transactions
- `Exec(), Query(), QueryRow()` can use any conncection in the `sql.DB` pool
- Even if we have consecutive calls to `Exec()`, it's not guaranteed to use the same connection
- Sometimes this is not acceptable
- If we lock a table in one connection, we must unlock it using the exact same connection to avoid deadlocks
- To guarantee that the same connection is used, we can wrap multiple statements in a transaction
- Calling the `Begin()` method on the connection pool represents in-progress transaction


- Transactions are also super-useful if we want to execute multiple SQL statements as a single atomic action.
- So long as we use the tx.Rollback() method in the event of any errors, the transaction ensures that either:
    - All statements are executed successfully; or
    - No statements are executed and the database remains unchanged.

### Prepared Statements
- `Exec(), Query(), QueryRow()` use prepared SQL statements, statements is re-prepared every time
- We can also create our own prepared statements using `DB.Prepare()` and reuse it later
- It becomes useful for complex queries with tens of thousands of records, because re-preprating such statements affects runtime cost
