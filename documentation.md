# rest_mysql
[![pypi version](https://img.shields.io/pypi/v/rest_mysql.svg)](https://pypi.org/project/rest_mysql) ![MIT License](https://img.shields.io/pypi/l/rest_mysql.svg)

Stand alone version of Record_MySQL from Rest-OC to facilitate updating code to
newer librairies. Since forking off of Rest-OC numerous improvements and bug
fixes have been done. See [releases](releases.md) for more info.

## Contents
- [Install](#install)
- [Module Configuration](#module-configuration)
- [Binary UUID and UUIDv4](#binary-uuid-and-uuidv4)
- [Table Configuration](#table-configuration)
- [Column Configuration](#column-configuration)

## Install

### Requires
rest_mysql requires python 3.10 or higher

### Install via pip
```bash
pip install rest_mysql
```

[ [top](#rest_mysql) / [contents](#contents) ]

## Binary UUID and UUIDv4
If you use `uuid` or `uuid4` in combination with the `binary` flag you will need
to add some functions to your database schema. See the
[Binary UUID and UUIDv4](README.md#binary-uuid-and-uuidv4) section in the
[README](README.md).

[ [top](#rest_mysql) / [contents](#contents) ]

## Module configuration

!!!!!!!
TODO!!!
!!!!!!!

[ [top](#rest_mysql) / [contents](#contents) ]

## Define
`rest_mysql` uses [define-oc](https://pypi.org/project/define-oc/) as the basis
for how to describe MySQL tables and their columns. It uses `define-oc`'s
`special()` method to look for `__sql__` sections at the `Tree` (Table) and
`Node` (Column / Field) level.

[ [top](#rest_mysql) / [contents](#contents) ]

## Table configuration
By setting the `__sql__` special section at the `Tree` level, we configure the
options for the table. Things like the primary key and indexes, or keeping track
of changes to table records.

```json
{
  "__name__": "my_table",
  "__sql__": {
    "changes": [ "user_id" ],
    "charset": "utf8mb4",
    "collate": "utf8mb4_0900_ai_ci",
    "create": [ "created", "updated", "name", "option" ],
    "db": "my_db",
    "indexes": {
      "name": { "unique": null }
    }
  },
  "_id": { "__type__": "tuuid" },
  "created": { "__type__": "timestamp" },
  "updated": { "__type__": "timestamp" },
  "name": { "__type__": "string", "__maximum__": 32 },
  "options": {
    "__array__": "unique",
    "__type__": "string",
    "__options__": [ "option1", "option2", "option3" ],
    "__sql__": { "json": true }
  }
}
```

### auto_primary
Defaults to `true`, which makes sense if [`primary`](#primary) is also left as
its default of **_id**. This also assumes `_id` is some sort of type that can
have an auto generated value. The only `Node` types that really make any sense
are, **int**, **tuuid**, **tuuid4**, **uint**, **uuid**, and **uuid4**. In the
case of **int** and **uint** `rest_mysql` uses the `AUTO_INCREMENT` feature, for
the others it uses a combination of MySQL variables and calling `UUID()`. Keep
in mind, that for UUID's you will need to call `UUID()` yourself if you bypass
`rest_mysql` in any way to insert new records, you may also have to replace
dashes and unhex as well if the [`__sql__.binary`](#__sql__binary) is set.

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### changes
Defaults to `false`.

Changes is used to create a separate `[table]_changes` table in the same
databases as `[table]`. This table is filled with the changes between the record
before and after a specific sql statement. In this way we can audit every single
change that has happened to a record from creation and beyond deletion.

If we set it to `true`, we simple get the changes from every stage including
INSERT, UPDATE, and DELETE. If we instead set it to a list of string(s), we
can define extra data, provided at runtime, to be stored alongside the changes.
For example, up above we had set
```json
    "changes": [ "user" ],
```
which means at any [`create`](#record-create), [`save`](#record-save), or
[`delete`](#record-delete) the user will be forced, by an exception, to pass
along a `dict` with the `user` key set.

```python
if not my_instance.create(
  changes = { 'user': 'my_user_id' }
):
```
Now we can not only keep track of changes, we can also see who made them. In
fact we'll know who created and potentially who deleted the record. Never again
will data, or a data trail, be missing.

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### charset
Defaults to **utf8mb4**, set it to whatever valid charset your server allows.

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### collate
Defaults to **utf8mb4_bin**, set it to whatever valid collate your server allows
with the chosen [`charset`](#charset).

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

## create
Due to an issue in python 2.7 that may or may no longer exist, JSON files
converted at runtime might not have the same order of keys in objects as their
original file. This meant that columns ended up being created in a completely
random order.

`create` fixes this by forcing you to state the order you want the columns to
be created in.

NOTE: Whether the primary key is stated in `create` or not makes no difference,
as the primay key will always be the first column in the table.

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### db
This is the name of the database schema your table will be created in and
accessed from. It default's to **test**.

If [`changes`](#changes) is set, the corresponding table will also be created in
this db.

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### engine
Defaults to **InnoDB**, set it to whatever valid engine your server allows.

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### host
Defaults to **records**. Corresponds to a section in `config.mysql.hosts` with
sql server connection information. See [Module Configuration](#module-configuration)
for more info.

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### indexes
Defaults to `{}`. Used to create indexes beyond the primary key. `indexes` Has 4
valid formats.

#### single column index
```json
  "column_name": null
```
becomes
```sql
  index `column_name` (`column_name`)
```

#### multi column index
```json
  "index_name": [ "column_name", "other_name" ]
```
becomes
```sql
  index `index_name` (`column_name`, `other_name`)
```

#### specific single column index
Allows **fulltext**, **index**, **spatial**, and **unique**
```json
  "column_name": { "fulltext": null }
```
becomes
```sql
  fulltext `column_name` (`column_name`)
```

#### specific multi column index
Allows **fulltext**, **index**, **spatial**, and **unique**
```json
  "index_name": {
    "unique": [ "column_name", "other_name" ]
  }
```
becomes
```sql
  unique `index_name` (`column_name`, `other_name`)
```

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### table
Defaults to the name of the `Tree`, but can be overwritten to whatever valid
table name you can use with your server.

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### primary
Defaults to **_id**. This is the column or columns which will comprise the primary
key of the table. To set it to a single column, pass a string
```json
    "primary": "name"
```
To set it to multiple columns, pass an array of strings
```json
    "primary": [ "name", "department" ]
```
To have no primary key on the table, set it to `false`
```json
    "primary": false
```

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### rev_field
Defaults to **_rev**, meaningless if [`revisions`](#revisions) is `false`.

Set to the name of the column you want to save the revision value in. Whatever
column you choose must be able to store at minimum a 34 character string. That
length increases by one for each digit added to the revisions. Meaning revision
strings 1 to 9 are 34 characters, 10 to 99 are 35 characters, 100 to 999
are 36, and so on. 

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

### revisions
Defaults to `false`. If set to `true` any attempt to [`save`](#record-save) a
record first checks if the revision string passed to `save` matches the one in
the database. Not matching means changes have occurred to the record between the
time the user started their changes and the time they attempted to save them.
If this is the case the save is rejected with a `RevisionException` in order to
avoid the user overwriting someone else's changes.

This flag is not used often due to the fact that `rest_mysql` only saves changes
in fields and not entire records, but in some cases the information might be so
sensitive that we can't risk any change being lost.

[ [top](#rest_mysql) / [contents](#contents) /
[table configuration](#table-configuration) ]

## Column configuration
Where it can, `rest_mysql` directly translates `config` `Node` types to SQL
types.\
For example
```json
  "count": { "__type__": "uint" }
```
becomes
```sql
  `count` integer unsigned not null
```
Other time's it's not as simple, so `rest_mysql` relies on some other settings
to decide.

### __minimum\_\_
In one edge case, where a string `Node`'s `__minimum__` is the same value as
the `__maximum__`, it triggers the use of `char` where it would normally be
`varchar`.\
For example
```json
  "pass_code": {
	"__minimum__": 5,
	"__maximum__": 5
  }
```
becomes
```sql
  `pass_code` char(5) not null
```

Most of the time `rest_mysql` leaves it to you to optimize how you see fit, but
this one case it only seems logical to default to `char` and force the user to
set [`__sql__.type`](#__sql__type)

[ [top](#rest_mysql) / [contents](#contents) /
[column configuration](#column-configuration) ]

### __maximum\_\_
In some cases, like strings, we don't need to explicitly set the
[type](#__sql__type) if the `Node` already has a maximum valid value. In these
cases `rest_mysql` can calculate the column width using the maximum.

#### base64 and string
If no [type](#__sql__type) is explicitely set, `rest_mysql` will require the maximum
value in order to calculate the column width. For example.
```json
  "name": {
    "__type__": "string",
    "__maximum__": 32
  }
```
becomes
```sql
  `name` varchar(32) not null
```

If you specifically want `text`, `mediumtext`, or `longtext` instead of
`varchar(n)`, you can use [type](#__sql__type), or you can set the maximum to their
exact respective maximums.

| sql | __maximum\_\_ |
| --------------| --- |
| text | 65535 |
| mediumtext | 16777215 |
| longtext | 4294967295 |

```json
  "description": {
    "__type__": "string",
    "__maximum__": 65535
  }
```
becomes
```sql
  `description` text not null
```
Where as
```json
  "description": {
    "__type__": "string",
    "__maximum__": 65534
  }
```
becomes
```sql
  `description` varchar(65534) not null
```

#### price
Prices are stored as decimals to avoid floating point errors. This requires
knowing the maximum value to be stored in order to know the max characters that
can be put into the column in the table.\
For example
```json
  "amount": {
    "__type__": "price",
	"__maximum__": "10000.00"
  }
```
would translate to
```sql
  `amount` decimal(7,2) not null
```
Where as
```json
  "amount": {
    "__type__": "price",
	"__maximum__": "9999.99"
  }
```
would translate to
```sql
  `amount` decimal(6,2) not null
```

[ [top](#rest_mysql) / [contents](#contents) /
[column configuration](#column-configuration) ]

### __optional\_\_
If the `__optional__` flag is set to `true`, then it's assumed the column is
allowed to be set to `NULL`.\
When we take the original example
```json
  "count": { "__type__": "uint" }
```
```sql
  `count` integer unsigned not null
```

We see that adding `__optional__`
```json
  "count": {
    "__type__": "uint",
	"__optional__": true
  }
```
removes "not null"
```sql
  `count` integer unsigned
```

[ [top](#rest_mysql) / [contents](#contents) /
[column configuration](#column-configuration) ]

### __options\_\_
When string types have `__options__`, those options are used to create an `enum`
instead of a `varchar`.\
For example
```json
  "language": {
    "__type__": "string",
	"__options__": [ "javascript", "python", "c++" ]
  }
```
becomes
```sql
  `language` enum('javascript', 'python', 'c++') not null
```

[ [top](#rest_mysql) / [contents](#contents) /
[column configuration](#column-configuration) ]

### __sql\_\_.binary
For the tuuid, tuuid4, uuid, and uuid4 types, we can add the `binary` flag so
that instead of using `char(32)` or `char(36)` we can use `binary(16)` saving
considerable space, especially on any indexes.\
For example
```json
  "_id": {
    "__type__": "tuuid",
    "__sql__": { "binary": true }
  }
```
becomes
```sql
  `_id` binary(16) not null
```
This won't affect the way you search or store data, `rest_mysql` will take care
of the conversion and you get the benefit of a smaller database without giving
up the benefits of unique keys.

[ [top](#rest_mysql) / [contents](#contents) /
[column configuration](#column-configuration) ]

### __sql\_\_.json
In some cases we want to store complex data in tables, but we don't have any
use for indexing or even filtering by that data so there's no real reason for
sub-tables and joins. In this case we can use `define-oc`'s `Array`, `Hash`,
and `Parent` types while still maintaining a flat table if we use the `json`
flag.\
For example
```json
  "title_by_locale": {
    "__hash__": {
      "__type__": "string",
      "__regex__": "^[a-z]{2}-[A-Z]{2}$"
    },
    "__type__": "string",
    "__sql__": { "json": true }
  }
```
becomes
```sql
  `title_by_locale` text not null
```
and `rest_mysql` takes care of validation as normal, then converts everything
under `title_by_locale` to JSON when storing, and from JSON when pulling data
out.
So now we can set
```json
  "title_by_locale": {
    "en-CA": "Hello, world!",
    "fr-CA": "Bonjour à tous!"
  }
```
and not have to worry about the fact this isn't valid in an SQL table.

[ [top](#rest_mysql) / [contents](#contents) /
[column configuration](#column-configuration) ]

### __sql\_\_.opts
`opts` is helpful when we want to set special options, or we want to override
options set by `rest_mysql` based on the standard `Node` settings.\
For example, if we had timestamps created at record creation and update
```json
  "created": {
    "__type__": "timestamp",
	"__optional__": true,
    "__sql__": {
      "opts": "not null default CURRENT_TIMESTAMP"
    }
  },
  "updated": {
    "__type__": "timestamp",
	"__optional__": true,
    "__sql__": {
      "opts": "not null default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP"
    }
  }
```
becomes
```sql
  `created` timestamp not null default CURRENT_TIMESTAMP,
  `updated` timestamp not null default CURRENT_TIMESTAMP
    on update CURRENT_TIMESTAMP
```
Despite the `Node` having the `__optional__` flag, we have overwritten it and
specifically told the server to use the current time as the default value and
update value.

[ [top](#rest_mysql) / [contents](#contents) /
[column configuration](#column-configuration) ]

### __sql\_\_.type
This is useful both to override `rest_mysql` when you don't want to the default
conversion from `Node` to SQL, and also sometimes necessary for those types
which require human decision making to set a value.

#### base64 and string
If for some reason you have a string that isn't options and doesn't have a
maximum for validation, you will be required to set the type.\
For example
```json
  "name": {
    "__type__": "string",
    "__sql__": { "type": "varchar(32)" }
  },
  "description": {
	"__type__": "string",
	"__sql__": { "type": "mediumtext" }
  }
```
becomes
```sql
  `name` varchar(32) not null,
  `description` text not null
```

#### decimal
The decimal type, rather than making assumptions, just flat out requires you
to set `__sql__.type`. Not doing so will cause an exception.
```json
  "percent": {
    "__type__": "decimal",
    "__minimum__": "0",
	"__maximum__": "100",
	"__sql__": { "type": "decimal(6,3)" }
}
```
becomes
```sql
  `percent` decimal(6,3) not null
```

#### override
Maybe you don't want a `varchar` even though you have no minimum string length,
or you don't want to use an emum. Whatever the case, you can override the type,
just make sure that it makes logical sense, because if you set a string to an
int or vice versa, `rest_mysql` won't be able to handle it and will likely raise
exceptions.

```json
  "name": {
    "__type__": "string",
    "__maximum__": 32,
    "__sql__": { "type": "char(32)" }
  },
  "location_type": {
    "__type__": "string",
    "__optional__": true,
    "__options__": [
      "Apartment", "Building",
      "Floor", "Suite",
      "Unit"
    ],
    "__sql__": { "type": "varchar(16)" }
  }
```
becomes
```sql
  `name` char(32) not null,
  `location_type` varchar(16)
```

[ [top](#rest_mysql) / [contents](#contents) /
[column configuration](#column-configuration) ]