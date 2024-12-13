# rest_mysql
Stand alone version of Record_MySQL from Rest-OC to facilitate updating code to newer librairies.

## Install

```bash
pip install rest_mysql
```

## Using

Instead of pulling Record_MySQL from RestOC as we did in the past, change any references to rest_mysql

Old:
```python
from RestOC import Record_MySQL
from RestOC.Record_MySQL import db_create, Record
from RestOC.Record_Base import register_type
```

New:
```python
from rest_mysql import Record_MySQL
from rest_mysql.Record_MySQL import db_create, Record
from rest_mysql.Record_Base import register_type
```