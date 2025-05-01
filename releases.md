# rest_mysql releases

## 1.2.4
- Switched from a single connection per host, to a pool of connections per host. The optional `maxconnections` argument on each host in `config.mysql.hosts` can be used to set exactly how many connections are available in the pool in order to support numerous workers using the module. By default `maxconnections` is 1, which provides the same behaviour as before.

## 1.2.3
- Added `Option` to the list of available `define_oc` types that can be converted to JSON.

## 1.2.2
- Fixed a bug where `Node` `json` types were being decoded as if they were `Parent`, `Hash`, or `Array` `json` types instead of just being returned as the expected JSON encoded string.

## 1.2.1
- Made Record_MySQL.add_host a DEPRECATED method, will be removed in future versions.
- Server host info is now pulled directly from `config.mysql.hosts` and no other setup is required to get up and running with `rest_mysql`.

## 1.2.0
- Updated `define_oc` to get access to `tuuid` and `tuuid4`
- Fixed a bug that happened after deleting records with complex primary keys.
- Added `__sql__.binary` flag for `uuid`, `uuid4`, `tuuid`, and `tuuid4` to store the values in binary format without changing how you interact with them.

## 1.1.5
- Now forcing of `__sql__.type` for 'decimal' fields.
- Now forcing of `__sql__.type` for 'price' fields that have no maximum value.
- Triggers can now be added to tables via the `triggers` section of the structure.
- Added `Record_Base.fields_set` method to set many fields at once.
- You can now use complex primary keys by passing them as a list to `primary`.

## 1.1.4
- Fixed POINT regex to allow negative values.

## 1.1.3
- Fix for 1.1.2 where commits were not being closed and tables were locking.

## 1.1.2
- Removed auto-committing on every statements and left each commit to be closed when the cursor is. This allowed refactoring `Commands.exec` to accept lists of SQL statements which are all run as a single commit.

## 1.1.1
- Added `Record_Base.keys()` class method.

## 1.1.0
- Adding typing to variables, arguments, and returns.
- Added ability to add **POINT** types.
- Removed the need for calling `timestamp_timezone()`.

## 1.0.3
- Fixed a bug where the `level` value was being passed to `ignore_missing`.

## 1.0.1
- Fixed a bug where 'ArrayNode' and 'HashNode' were still referenced instead of the new 'Array' and 'Hash' class types.

## 1.0.0
- Copied the **Record_Base** and **Record_MySQL** modules out of `REST-OC` and updated them to use `define-oc` and `tools-oc`.