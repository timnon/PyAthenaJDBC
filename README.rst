PyAthenaJDBC
============

PyAthenaJDBC is a python `DB API 2.0 (PEP
249) <https://www.python.org/dev/peps/pep-0249/>`__ compliant wrapper
for `Amazon Athena JDBC
driver <http://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html>`__.

Requirements
------------

-  Python

    -  CPython 2.6, 2,7, 3,4, 3.5

-  Java

    -  Java 7+

Installation
------------

.. code:: bash

    $ pip install PyAthenaJDBC

Credential
----------

Support `AWS CLI credentials
configuration <http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html>`__.

Credential Files
~~~~~~~~~~~~~~~~

~/.aws/credentials

.. code:: cfg

    [default]
    aws_access_key_id=YOUR_ACCESS_KEY_ID
    aws_secret_access_key=YOUR_SECRET_ACCESS_KEY

~/.aws/config

.. code:: cfg

    [default]
    region=us-west-2
    output=json

Environment variables
~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    $ export AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID
    $ export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_ACCESS_KEY
    $ export AWS_DEFAULT_REGION=us-west-2

Additional environment variable:

.. code:: bash

    $ export AWS_ATHENA_S3_STAGING_DIR=s3://YOUR_S3_BUCKET/path/to/

Usage
-----

Basic usage:

.. code:: python

    from pyathenajdbc import connect

    conn = connect(s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/')
    try:
        with conn.cursor() as cursor:
            cursor.execute("""
            SELECT * FROM test_pyathena_jdbc.one_row
            """)
            print(cursor.description)
            print(cursor.fetchall())
    finally:
        conn.close()

Cursor iteration:

.. code:: python

    from pyathenajdbc import connect

    conn = connect(s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/')
    try:
        with conn.cursor() as cursor:
            cursor.execute("""
            SELECT * FROM test_pyathena_jdbc.many_rows LIMIT 10
            """)
            for row in cursor:
                print(row)
    finally:
        conn.close()

Query with parameter:

.. code:: python

    from pyathenajdbc import connect

    conn = connect(s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/')
    try:
        with conn.cursor() as cursor:
            cursor.execute("""
            SELECT col_int FROM test_pyathena_jdbc.one_row_complex where col_int = {0}
            """, 2147483647)
            print(cursor.fetchall())

            cursor.execute("""
            SELECT col_string FROM test_pyathena_jdbc.one_row_complex where col_string = {param}
            """, param='a string')
            print(cursor.fetchall())
    finally:
        conn.close()

Pandas DataFrame:

.. code:: python

    import contextlib
    from pyathenajdbc import connect
    from pyathenajdbc.util import as_pandas

    with contextlib.closing(
            connect(s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/')) as conn:
        with conn.cursor() as cursor:
            cursor.execute("""
            SELECT * FROM test_pyathena_jdbc.many_rows
            """)
            df = as_pandas(cursor)
    print(df.describe())

Testing
-------

Depends on the following environment variables:

.. code:: bash

    $ export AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID
    $ export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_ACCESS_KEY
    $ export AWS_DEFAULT_REGION=us-west-2
    $ export AWS_ATHENA_S3_STAGING_DIR=s3://YOUR_S3_BUCKET/path/to/

Run test:

.. code:: bash

    $ pip install pytest
    $ py.test

Run test multiple Python versions:

.. code:: bash

    $ pip install tox
    $ pyenv local 2.6.9 2.7.12 3.4.5 3.5.2
    $ tox