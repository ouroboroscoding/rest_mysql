# rest_mysql
[![pypi version](https://img.shields.io/pypi/v/rest_mysql.svg)](https://pypi.org/project/rest_mysql) ![MIT License](https://img.shields.io/pypi/l/rest_mysql.svg)

Stand alone version of Record_MySQL from Rest-OC to facilitate updating code to
newer librairies. Since forking off of Rest-OC numerous improvements and bug
fixes have been done. See [releases](releases.md) for more info.

## Contents
- [Install](#install)
- [Binary UUID and UUIDv4](#binary-uuid-and-uuidv4)
- [Table Configuration](#table-configuration)
- [Column Configuration](#column-configuration)

## Install

### Requires
rest_mysql requires python 3.10 or higher

### Install via pip
```console
foo@bar:~$ pip install rest_mysql
```

[ [top](#rest_mysql) / [contents](#contents) ]

## Binary UUID and UUIDv4
If you use `uuid` or `uuid4` in combination with the `binary` flag you will need
to add some functions to your database schema. See the
[Binary UUID and UUIDv4](README.md#binary-uuid-and-uuidv4) section in the
[README](README.md).

[ [top](#rest_mysql) / [contents](#contents) ]

## Define
`rest_mysql` uses [define-oc](https://pypi.org/project/define-oc/) as the basis
for how to describe MySQL tables and their columns. It uses `define-oc`'s
`special()` method to look for `__sql__` sections at the `Tree` (Table) and
`Node` (Column / Field) level.

[ [top](#rest_mysql) / [contents](#contents) ]

## Table configuration


### auto_primary

### 

## Column configuration
Where it can, `rest_mysql` directly translates `config` `Node` types to SQL
ypes. For example
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

[ [top](#rest_mysql) / [contents](#contents) /
[column configuration](#column-configuration) ]

### __maximum\_\_
In some cases, like strings, we don't need to explicitly set the [type](#sqltype)
if the `Node` already has a maximum valid value. In these cases
`rest_mysql` can calculate the column width using the maximum.

#### base64 and string
If no [type](#sqltype) is explicitely set, `rest_mysql` will require the maximum
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
`varchar(n)`, you can use [type](#sqltype), or you can set the maximum to their
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