# gomigrate

[![Build Status](https://travis-ci.org/motivq/gomigrate.svg?branch=master)](https://travis-ci.org/motivq/gomigrate)

A SQL database migration toolkit in Golang.

This is a fork of https://github.com/DavidHuie/gomigrate that adds support for
reading migration files embedded directly within the built executable, or from
any other source available via the `fs.FS` interface present in Go 1.16 and up.
This simplifies deployment by removing the need to install and manage a
separate migrations directory.

The previous file-based API also remains supported, even though the
implementation now uses `fs.FS` internally.

## Supported databases

- PostgreSQL
- MariaDB
- MySQL
- Sqlite3

## Usage

First import the Go 1.16 embed package and this package:

```go
import (
	"embed"
	"github.com/motivq/gomigrate"
)
```

Given a `database/sql` database connection to a PostgreSQL database, `db`,
and a directory named `migrations` in the same directory as the source code,
create a migrator that reads files within the embedded `migrations` directory:

```go
//go:embed migrations
var migrationsDirFS embed.FS // files will be embedded under path "migrations/"

// make an FS to access the files without the "migrations/" path prefix
migrationsFS, err := fs.Sub(migrationsDirFS, "migrations")
if err != nil {
	log.Fatal(err)
}
migrator, _ := gomigrate.NewMigratorFS(db, gomigrate.Postgres{}, migrationsFS)
```

You may also specify a logger, such as logrus:

```go
migrator, _ := gomigrate.NewMigratorFSWithLogger(db, gomigrate.Postgres{}, migrationsFS, logrus.New())
```

To migrate the database, run:

```go
err := migrator.Migrate()
```

To rollback the last migration, run:

```go
err := migrator.Rollback()
```

## Migration files

Migration files need to follow a standard format and must be present
in the same directory. Given "up" and "down" steps for a migration,
create a file for each by following this template:

```
{{ id }}_{{ name }}_{{ "up" or "down" }}.sql
```

For a given migration, the `id` and `name` fields must be the same.
The id field is an integer that corresponds to the order in which
the migration should run relative to the other migrations.

`id` should not be `0` as that value is used for internal validations.

Note that in the example above of embedding migrations, Go will automatically
skip filenames beginning with a period or underscore. This is typically useful,
but could be overridden if necessary by using a wildcard glob pattern.

### Example

If I'm trying to add a "users" table to the database, I would create
the following two files:

#### 1_add_users_table_up.sql

```
CREATE TABLE users();
```

#### 1_add_users_table_down.sql
```
DROP TABLE users;
```

## Requirements

Go 1.16 or higher.


## Copyright

Copyright © 2014-2021 David Huie and Andrew Shearer. Based on
https://github.com/DavidHuie/gomigrate by David Huie. See LICENSE.txt
for further details.
