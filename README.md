# PML-SQLITEORM

ORM between PML and SQLite3 database via sqlite3.exe CLI

## Prerequisites

You require to put [sqlite3.exe](https://sqlite.org/index.html) in any directory defined by the `PATH` environment variable. If it is hard to prepare it for everyone who uses PML macros, you can put `sqlite3.exe` to the `bin` directory to use it instead.

## Installation

Clone this repository and put it in a directory defined by the `PMLLIB` environment variable.

```bat
git clone https://github.com/k-awata/pml-sqliteorm.git <Your-PMLLIB-Folder>/sqliteorm
```

## Usage

### Open a database file

`.sqliteorm(!dbFile is STRING)`

```sql
!db = object SQLITEORM('%TEMP%\sqlite3.db')

-- Get the database as FILE object
q var !db.dbFile
```

### Select data

`.Select(!table is STRING, !columns is STRINGMAP, !where is STRINGMAP) is RECORDSET`

`.Select(!table is STRING, !columns is STRING, !where is STRINGMAP) is RECORDSET`

```sql
-- Columns with specified name
$* SELECT "id" AS 'col1', "name" AS 'col2', "gender" AS 'col3' FROM "users" WHERE "gender" = 'Female';
!recordset = !db.Select( $
    'users', $
    !!stringmap('{"col1": "\"id\"", "col2": "\"name\"", "col3": "\"gender\""}'), $
    !!stringmap('{"gender": "Female"}') $
)

-- Columns from raw string
$* SELECT "id", "name", "gender", "updated_at", "deleted" FROM "users" WHERE "gender" = 'Female';
!recordset = !db.Select( $
    'users', $
    '"id", "name", "gender", "updated_at", "deleted"', $
    !!stringmap('{"gender": "Female"}') $
)

-- Foreach loop
do !record values !recordset.StringMaps()
    skip if !record.GetBoolean('deleted')
    q var !record.Get('name')
enddo

-- First record
q var !recordset.First().GetDateTime('updated_at')

-- Values of specified column
q var !recordset.Column('id')
```

### Insert data

`.Insert(!table is STRING, !record is STRINGMAP)`

`.Insert(!table is STRING, !records is RECORDSET)`

```sql
$* INSERT INTO "users" ("id","name","gender") VALUES ('1','Alice','Female');
!db.Insert('users', !!stringmap('{"id": 1, "name": "Alice", "gender": "Female"}'))

-- Bulk insert
!data = object RECORDSET()
!data.Append(!!stringmap('{"id": 2, "name": "Bob",   "gender": "Male"  }'))
!data.Append(!!stringmap('{"id": 3, "name": "Carol", "gender": "Female"}'))
!data.Append(!!stringmap('{"id": 4, "name": "Dan",   "gender": "Male"  }'))
!db.Insert('users', !data)
```

### Replace data

`.Replace(!table is STRING, !record is STRINGMAP)`

`.Replace(!table is STRING, !records is RECORDSET)`

```sql
$* REPLACE INTO "users" ("id","name","gender") VALUES ('3','Charlie','Male');
!db.Replace(!!stringmap('{"id": 3, "name": "Charlie", "gender": "Male"}')
```

### Update data

`.Update(!table is STRING, !record is STRINGMAP, !where is STRINGMAP)`

```sql
!data = object STRINGMAP()
!data.SetDateTime('updated_at', object DATETIME(2020, 1, 23, 4, 56, 42))

$* UPDATE "users" SET "updated_at" = '2020-01-23 04:56:42' WHERE "id" = '1';
!db.Update('users', !data, !!stringmap('{"id": 1}'))
```

### Delete data

`.Delete(!table is STRING, !where is STRINGMAP)`

```sql
$* DELETE FROM "users" WHERE "id" = '1';
!db.Delete('users', !!stringmap('{"id": 1}')
```

### Get whether any data exists

`.Exists(!table is STRING, !where is STRINGMAP) is BOOLEAN`

```sql
$* SELECT EXISTS(SELECT * FROM "users" WHERE "id" = '1');
q var !db.Exists('users', !!stringmap('{"id": 1}'))
```

### Get total number of data rows

`.Count(!table is STRING, !where is STRINGMAP) is REAL`

```sql
$* SELECT COUNT(*) FROM "users" WHERE "gender" = 'Male';
q var !db.Count('users', !!stringmap('{"gender": "Male"}'))
```

### Execute raw SQL

`.Execute(!sql is STRING)`

`.Execute(!sql is ARRAY)`

`.Execute(!sql is FILE)`

`.Query(!sql is STRING) is RECORDSET`

`.Query(!sql is ARRAY) is RECORDSET`

`.Query(!sql is FILE) is RECORDSET`

```sql
-- Raw SQL
!db.Execute(|
    CREATE TABLE "users" (
        "id"         INTEGER NOT NULL,
        "name"          TEXT NOT NULL,
        "gender"        TEXT NOT NULL,
        "updated_at"    TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
        "deleted"    INTEGER NOT NULL DEFAULT 0,
        PRIMARY KEY("id")
    );
|)

-- Raw SQL from a file
q var !db.Query(object FILE('%TEMP%\query.sql'))
```

## Tests

To run the test cases, use [PML Unit](https://github.com/PoByBolek/PmlUnit) on Everything3D 2.1.
