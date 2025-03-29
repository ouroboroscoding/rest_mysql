# rest_mysql
[![pypi version](https://img.shields.io/pypi/v/rest_mysql.svg)](https://pypi.org/project/rest_mysql) ![MIT License](https://img.shields.io/pypi/l/rest_mysql.svg)

Stand alone version of Record_MySQL from Rest-OC to facilitate updating code to
newer librairies. Since forking off of Rest-OC numerous improvements and bug
fixes have been done. See [releases](releases.md) for more info.

## Contents
- [Install](#install)
- [Binary UUID and UUIDv4](#binary-uuid-and-uuidv4)
- [Configuration]()

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

