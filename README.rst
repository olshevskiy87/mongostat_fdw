#############
mongostat_fdw
#############

Mongo database and collection statistics foreign data wrapper for PostgreSQL written in python.

************
dependencies
************

* `pymongo <https://pypi.python.org/pypi/pymongo>`__
* `multicorn <http://multicorn.org/#idinstallation>`__

************
installation
************

1. install python module

 * from sources (bitbucket)

    ::

        $ git clone https://bitbucket.org/olshevskiy87/mongostat_fdw.git
        $ cd mongostat_fdw
        $ python setup.py install

 * using pip

    ::

        $ pip install mongostat_fdw

2. create extension "multicorn" in postgresql (e.g. using psql)

    ::

        $$ create extension multicorn;

3. create foreign servers

    ::

        $$ CREATE SERVER mongostat_fdw_db
           FOREIGN DATA WRAPPER multicorn
           OPTIONS (
               wrapper 'mongostat_fdw.MongoDBStatFDW'
           );

    ::

        $$ CREATE SERVER mongostat_fdw_coll
           FOREIGN DATA WRAPPER multicorn
           OPTIONS (
               wrapper 'mongostat_fdw.MongoCollStatFDW'
           );

4. create foreign tables

    ::

        $$ CREATE FOREIGN TABLE mongo_db_stat (
            "avgObjSize" NUMERIC,
            collections INT,
            "dataFileVersion" JSONB, -- JSONB for Postgres 9.4+ or JSON, or (at least) TEXT
            "extentFreeList" JSONB,  -- Mongo 3.0.0+
            "dataSize" NUMERIC,
            db TEXT,
            "fileSize" NUMERIC,
            "indexSize" NUMERIC,
            indexes INT,
            "nsSizeMB" BIGINT,
            "numExtents" INT,
            objects INT,
            ok NUMERIC,
            "storageSize" NUMERIC
        ) SERVER mongostat_fdw_db OPTIONS (
            -- uri 'mongodb://127.0.0.1:27017',
            host '127.0.0.1',
            port '27017',
            db 'test'
        );

    ::

        $$ CREATE FOREIGN TABLE mongo_coll_stat (
            "avgObjSize" NUMERIC,
            count INT,
            "indexSize" JSONB,
            "lastExtentSize" NUMERIC,
            nindexes INT,
            ns TEXT,
            "numExtents" INT,
            ok NUMERIC,
            "paddingFactor" NUMERIC,
            size NUMERIC,
            "storageSize" NUMERIC,
            "systemFlags" INT,
            "totalIndexSize" NUMERIC,
            "userFlags" INT
        ) SERVER mongostat_fdw_coll OPTIONS (
            db 'test'
        );

*****
usage
*****

* get "test" database statistics

::

    $$ select db, "fileSize", "dataSize", "avgObjSize", indexes, "dataFileVersion"
       from mongo_db_stat;

      db   | fileSize | dataSize |  avgObjSize   | indexes |     dataFileVersion
    -------+----------+----------+---------------+---------+--------------------------
     local | 67108864 |     2840 | 405.714285714 |       1 | {"major": 4, "minor": 5}
     admin |        0 |        0 |           0.0 |       0 | {}
    (2 rows)

* get "test" database collections statistics

::

    $$ select ns as tbl_name, size, "storageSize", count
       from mongo_coll_stat;

          tbl_name       | size | storageSize | count
    ---------------------+------+-------------+-------
     test.system.indexes |   72 |        4096 |     1
     test.test_coll      |  344 |        4096 |     7
    (2 rows)

**************
external links
**************

* `PostgreSQL foreign data wrappers <https://wiki.postgresql.org/wiki/Foreign_data_wrappers>`__
* `Multicorn <http://multicorn.org>`__ - Postgres extension that allows to make FDW with python language
* `MongoDB <https://www.mongodb.com>`__ - a high performance document-oriented DBMS with automatic scaling
* `MongoDB dbStats command <https://docs.mongodb.com/manual/reference/command/dbStats/>`__
* `pymongo <https://pypi.python.org/pypi/pymongo>`__ - python distribution for working with MongoDB

*******
license
*******

Copyright (c) 2016 Dmitriy Olshevskiy. MIT LICENSE.

See LICENSE.txt for details.
