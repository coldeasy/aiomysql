aiomysql
========
.. image:: https://travis-ci.org/jettify/aiomysql.svg
    :target: https://travis-ci.org/jettify/aiomysql
.. image:: https://coveralls.io/repos/jettify/aiomysql/badge.png?branch=master
    :target: https://coveralls.io/r/jettify/aiomysql?branch=master

aiomysql is a library for accessing a MySQL database from the asyncio_
(PEP-3156/tulip) framework. This library is fork of Tornado-MySQL_ and
based on PyMySQL_ .


Basic Example
-------------

.. code:: python

    import asyncio
    import aiomysql

    loop = asyncio.get_event_loop()

    @asyncio.coroutine
    def test_example():
        conn = yield from aiomysql.connect(host='127.0.0.1', port=3306,
                                           user='root', passwd='', db='mysql',
                                           loop=loop)

        cur = conn.cursor()
        yield from cur.execute("SELECT Host,User FROM user")
        print(cur.description)
        r = yield from cur.fetchall()
        print(r)
        yield from cur.close()
        conn.close()

    loop.run_until_complete(test_example())


Connection Pool
---------------
Connection pooling ported from aiopg_ :

.. code:: python

    import asyncio
    import aiomysql


    loop = asyncio.get_event_loop()


    @asyncio.coroutine
    def test_example():
            pool = yield from aiomysql.create_pool(host='127.0.0.1', port=3306,
                                                   user='root', passwd='',
                                                   db='mysql', loop=loop)
            with (yield from pool) as conn:
                cur = conn.cursor()
                yield from cur.execute("SELECT 10")
                # print(cur.description)
                (r,) = yield from cur.fetchone()
                assert r == 10
            pool.close()
            yield from pool.wait_closed()

    loop.run_until_complete(test_example())


Example of SQLAlchemy optional integration
-------------------------------------------
Sqlalchemy support has been ported from aiopg_:

.. code:: python

   import asyncio
   from aiomysql.sa import create_engine
   import sqlalchemy as sa


   metadata = sa.MetaData()

   tbl = sa.Table('tbl', metadata,
       sa.Column('id', sa.Integer, primary_key=True),
       sa.Column('val', sa.String(255)))


   @asyncio.coroutine
   def go():
       engine = yield from create_engine(user='root',
                                         db='aiomysql',
                                         host='127.0.0.1',
                                         password='')

       with (yield from engine) as conn:
           yield from conn.execute(tbl.insert().values(val='abc'))

           res = yield from conn.execute(tbl.select())
           for row in res:
               print(row.id, row.val)


   asyncio.get_event_loop().run_until_complete(go())


TODO
----
* refactor connection closing
* implement ssl transport support
* rethink autocommit in tests
* documentation
* implement echo like in aiopg
* bring back loggers like in pymysql

Requirements
------------

* Python_ 3.3+
* asyncio_ or Python_ 3.4+
* PyMySQL_


.. _Python: https://www.python.org
.. _asyncio: http://docs.python.org/3.4/library/asyncio.html
.. _aiopg: https://github.com/aio-libs/aiopg
.. _PyMySQL: https://github.com/PyMySQL/PyMySQL
.. _Tornado-MySQL: https://github.com/PyMySQL/Tornado-MySQL
