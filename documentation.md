# rest_mysql
[![pypi version](https://img.shields.io/pypi/v/rest_mysql.svg)](https://pypi.org/project/rest_mysql) ![MIT License](https://img.shields.io/pypi/l/rest_mysql.svg)

Stand alone version of Record_MySQL from Rest-OC to facilitate updating code to
newer librairies. Since forking off of Rest-OC numerous improvements and bug
fixes have been done. See [releases](releases.md) for more info.

## Contents
- [Install](#install)
- [Binary UUID and UUIDv4](#binary-uuid-and-uuidv4)
- [Module Configuration](#module-configuration)
- [Table Configuration](#table-configuration)
- [Column Configuration](#column-configuration)
- [Using](#using)
- [Exceptions](#exceptions)

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

### Table configuration
By setting the `__sql__` special section at the `Tree` level, we configure the
options for the table. Things like the primary key and indexes, or keeping track
of changes to table records.

```json
{
  "__sql__": {
    "changes": [ "user" ],
    "charset": "utf8mb4",
    "collate": "utf8mb4_0900_ai_ci",
    "create": [ "_created", "_updated", "name", "options" ],
    "db": "my_db",
    "indexes": {
      "name": { "unique": null }
    }
  }
}
```

#### auto_primary
Defaults to `true`, which makes sense if [`primary`](#primary) is also left as
its default of **_id**. This also assumes `_id` is some sort of type that can
have an auto generated value. The only `Node` types that really make any sense
are, **int**, **tuuid**, **tuuid4**, **uint**, **uuid**, and **uuid4**.

In the case of **int** and **uint** `rest_mysql` uses the `AUTO_INCREMENT`
feature

For **tuuid**, **tuuid4**, **uuid**, and **uuid4**, it uses a combination of
MySQL variables and calling `UUID()`. Keep in mind, that for UUID's you will
need to call `UUID()` yourself if you bypass `rest_mysql` in any way to insert
new records. If you also use the [`__sql__.binary`](#__sql__binary) flag you
also have to replace dashes and unhex as well.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### changes
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
[`delete`](#record-delete) the user will be forced to pass along a `dict` with
the `user` key set.

```python
if not my_instance.create(
  changes = { 'user': 'my_user_id' }
):
```
Now we can not only keep track of changes, we can also see who made them. In
fact we'll know who created and potentially who deleted the record. Never again
will data, or a data trail, be missing.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### charset
Defaults to **utf8mb4**, set it to whatever valid charset your server allows.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### collate
Defaults to **utf8mb4_bin**, set it to whatever valid collate your server allows
with the chosen [`charset`](#charset).

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### create
Due to an issue in python 2.7 that may or may no longer exist, JSON files
converted at runtime might not have the same order of keys in objects as their
original file. This meant that columns ended up being created in a completely
random order.

`create` fixes this by forcing you to state the order you want the columns to
be created in.

NOTE: Whether the primary key is stated in `create` or not makes no difference,
as the primay key will always be the first column in the table.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### db
This is the name of the database schema your table will be created in and
accessed from. It default's to **test**.

If [`changes`](#changes) is set, the corresponding table will also be created in
this db.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### engine
Defaults to **InnoDB**, set it to whatever valid engine your server allows.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### host
Defaults to **records**. Corresponds to a section in `config.mysql.hosts` with
sql server connection information. See [Module Configuration](#module-configuration)
for more info.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### indexes
Defaults to `{}`. Used to create indexes beyond the primary key. `indexes` Has 4
valid formats.

##### single column index
```json
  "column_name": null
```
becomes
```sql
  index `column_name` (`column_name`)
```

##### multi column index
```json
  "index_name": [ "column_name", "other_name" ]
```
becomes
```sql
  index `index_name` (`column_name`, `other_name`)
```

##### specific single column index
Allows **fulltext**, **index**, **spatial**, and **unique**
```json
  "column_name": { "fulltext": null }
```
becomes
```sql
  fulltext `column_name` (`column_name`)
```

##### specific multi column index
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

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### table
Defaults to the name of the `Tree`, but can be overwritten to whatever valid
table name you can use with your server.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### primary
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

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### rev_field
Defaults to **_rev**, meaningless if [`revisions`](#revisions) is `false`.

Set to the name of the column you want to save the revision value in. Whatever
column you choose must be able to store at minimum a 34 character string. That
length increases by one for each digit added to the revisions. Meaning revisions
1 to 9 are 34 characters, 10 to 99 are 35 characters, 100 to 999 are 36, and so
on.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

#### revisions
Defaults to `false`. If set to `true` any attempt to [`save`](#record-save) a
record first checks if the revision string passed to `save` matches the one in
the database. Not matching means changes have occurred to the record between the
time the user started their changes and the time they attempted to save them.
If this is the case the save is rejected with a `RevisionException` in order to
avoid the user overwriting someone else's changes.

This flag is not used often due to the fact that `rest_mysql` only saves changes
in fields and not entire records, but in some cases the information might be so
sensitive that we can't risk any change being lost.

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[table configuration](#table-configuration) ]

### Column configuration
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

#### __minimum\_\_
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

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[column configuration](#column-configuration) ]

#### __maximum\_\_
In some cases we don't need to explicitly set the [type](#__sql__type) if the
`Node` already has a maximum valid value. In these cases `rest_mysql` can
calculate the column width using the maximum.

##### base64 and string
If no [type](#__sql__type) is explicitely set, `rest_mysql` will require the
maximum value in order to calculate the column width. For example.
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

##### price
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

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[column configuration](#column-configuration) ]

#### __optional\_\_
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

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[column configuration](#column-configuration) ]

#### __options\_\_
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

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[column configuration](#column-configuration) ]

#### __sql\_\_.binary
For the **tuuid**, **tuuid4**, **uuid**, and **uuid4** types, we can add the
`binary` flag so that instead of `char(32)` or `char(36)` we use `binary(16)`,
saving considerable space on any indexes.\
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

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
[column configuration](#column-configuration) ]

#### __sql\_\_.json
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

NOTE: while `rest_mysql` will take care of converting the object to and from
JSON, it is not clever enough to know when you change something below the
parent's name. If you need to change a value in the table that's store as JSON,
you either need to set the entire value, or set `replace` when calling
[`Record.save`](#record-save)

BAD!
```python
from my_record import MyRecord
record = MyRecord.get('some_id')
record['title_by_locale']['en-CA'] = 'Good morning!'
record.save()
```
Here `rest_mysql` won't be notified `title_by_locale` changed, and so the call
to `save()` will return `False` here because no other columns are changed.
However if we are changing multiple columns, `save()` can still return `True`
even though `title_by_locale` won't have been updated in the DB.

Avoid hard to track bugs and always set JSON columns at their top level. Use
`combine` if you have to.

```python
from tools import combine
from my_record import MyRecord
record = MyRecord.get('some_id')
record['title_by_locale'] = combine(
  record['title_by_locale'], {
    'en-CA': 'Good morning!'
  }
)
```
Or just replace the value altogether if you can
```python
record['title_by_locale'] = {
  'en-CA': 'Good morning!',
  'fr-CA': 'Bonjour à tous!'
}
```

[ [top](#rest_mysql) / [contents](#contents) / [define](#define) /
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

## Using

### Everything in one file / dict
If you don't care what info you store in the JSON, or there is no JSON and you
only use the Tree for server side validation, you can just put everything into
one object / dict and load it as is.

`my_record.json`
```json
{
  "__name__": "my_record",
  "_id": {
    "__type__": "tuuid",
    "__optional__": true,
    "__sql__": {
      "binary": true
    }
  },
  "_created": {
    "__type__": "timestamp",
    "__optional__": true,
    "__sql__": {
      "opts": "default current_timestamp"
    }
  },
  "name": {
    "__type__": "string",
    "__maximum__": 32
  },
  "options": {
    "__array__": "unique",
    "__type__": "string",
    "__options__": [
      "option1",
      "option2",
      "option3"
    ],
    "__sql__": {
      "json": true
    }
  },
  "__sql__": {
    "collate": "utf8mb4_0900_ai_ci",
    "create": [
      "_created",
      "name",
      "options"
    ],
    "db": "my_db",
    "indexes": {
      "name": {
        "unique": null
      }
    }
  }
}
```

`my_record.py`
```python
from define import Tree
from rest_mysql import Record
class MyRecord(Record):
  _conf = Record.generate_config(
    Tree.from_file('my_record.json')
  )
  def config(cls):
    return cls._conf
```

[ [top](#rest_mysql) / [contents](#contents) / [using](#using) ]

### Separate define and mysql settings
If you're sharing your files with the UI side, it's good practice not to put
the SQL settings in the shared file. Instead we can keep the define file clean,
and use the built in `extend` argument of all constructors and the from_file
method, for just this purpose.

`my_record.json`
```json
{
  "__name__": "my_record",
  "_id": {
    "__type__": "tuuid",
    "__optional__": true
  },
  "_created": {
    "__type__": "timestamp",
    "__optional__": true
  },
  "name": {
    "__type__": "string",
    "__maximum__": 32
  },
  "options": {
    "__array__": "unique",
    "__type__": "string",
    "__options__": [
      "option1",
      "option2",
      "option3"
    ]
  }
}
```

`my_record.py`
```python
from define import Tree
from rest_mysql import Record
class MyRecord(Record):
  _conf = Record.generate_config(
    Tree.from_file('my_record.json', {
      '_id': { '__sql__': {
        'binary': True
      } },
      '_created': { '__sql__': {
        'opts': 'default current_timestamp'
      } },
      'options': { '__sql__': {
        'json': True
      } },
      '__sql__': {
        'collate': 'utf8mb4_0900_ai_ci',
        'create': [ '_created', 'name', 'options' ],
        'db': 'my_db',
        'indexes': {
          'name': { 'unique': null }
        }
      }
    })
  )
  def config(cls):
    return cls._conf
```

[ [top](#rest_mysql) / [contents](#contents) / [using](#using) ]

### Accessing and manipulating records

```python
from sys import exit
from my_record import MyRecord
record = MyRecord({
  'name': 'Phoenix',
  'options': [ 'option1', 'option3' ]
}).create(changes)

if not record.create():
  exit(1)

record2 = MyRecord(
  MyRecord.get(
    record['_id'],
    raw = [ 'name', 'options' ]
  )
)
record2['name'] = 'copy_of_%s' % record2['name']
if not record2.create():
  exit(1)

records = MyRecord.filter({
  'name': { 'like': '%Phoenix' }
}, orderby = 'name')

for o in records:
  o.delete()
```

## Record_MySQL

### db_create

### db_drop

### verbose

### Commands

#### escape

#### execute

#### insert

#### select

## Record

### class methods
add_changes, count, create_many, create_now, delete_get, escape, exists, filter,
generate_changes, generate_config, get, get_changes, keys, process_record,
process_field, process_value, search, struct, table_create, table_drop,
table_name, triggers_create, triggers_drop, triggers_recreate, update_field,
uuid

### instance methods
changed, changes, create, delete, field_delete, field_get, field_set,
fields_set, record, save

## Exceptions
`rest_mysql` has two `Exception` types, [`DuplicateException`](#duplicateexception)
and [`RevisionException`](#revisionexception).

### DuplicateException
Raised when any `create` or `save` call results in a duplicate key error from
MySQL. `arg[0]` is the key that's been duplicated, and `arg[1]` is the name of
the primary key or unique index.
```python
from rest_mysql import DuplicateException
from my_record import MyRecord
try:
  o = MyRecord({ """ some data """ })
  o.create()
except DuplicateException as e:
  print('Index "%s" already contains "%s"' % (e.args[1], e.args[0]))
```

[ [top](#rest_mysql) / [contents](#contents) / [exceptions](#exceptions) ]

### RevisionException
Raised when someone attempts to save to a record using an old revision key.
`arg[0]` is the primary key of the record.
```python
from rest_mysql import RevisionException
from my_record import MyRecord
try:
  o = MyRecord({ """ some old data """ })
  o.save()
except RevisionException as e:
  print('Record "%s" has been updated. Please refresh your data.' % e.args[0])
```

[ [top](#rest_mysql) / [contents](#contents) / [exceptions](#exceptions) ]
