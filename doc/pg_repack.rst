pg_repack -- Reorganize tables in PostgreSQL databases without any locks
========================================================================

.. contents::
    :depth: 1
    :backlinks: none

Synopsis
--------

::

    pg_repack [OPTION]... [DBNAME]

The following options can be specified in ``OPTIONS``. See also Options_ for
details.

Options:
  -a, --all                 repack all databases
  -n, --no-order            do vacuum full instead of cluster
  -o, --order-by=COLUMNS    order by columns instead of cluster keys
  -t, --table=TABLE         repack specific table only
  -T, --wait-timeout=SECS   timeout to cancel other backends on conflict
  -Z, --no-analyze          don't analyze at end

Connection options:
  -d, --dbname=DBNAME       database to connect
  -h, --host=HOSTNAME       database server host or socket directory
  -p, --port=PORT           database server port
  -U, --username=USERNAME   user name to connect as
  -w, --no-password         never prompt for password
  -W, --password            force password prompt

Generic options:
  -e, --echo                echo queries
  -E, --elevel=LEVEL        set output message level
  --help                    show this help, then exit
  --version                 output version information, then exit


Description
-----------

pg_repack is an utility program to reorganize tables in PostgreSQL databases.
Unlike clusterdb_, it doesn't block any selections and updates during
reorganization.

pg_repack is a fork of the previous pg_reorg_ project. It was founded to
gather the bug fixes and new development ideas that the slow pace of
development of pg_reorg was struggling to satisfy.

You can choose one of the following methods to reorganize:

* Online CLUSTER (ordered by cluster index)
* Ordered by specified columns
* Online VACUUM FULL (packing rows only)

NOTICE:

* Only superusers can use the utility.
* Target table must have PRIMARY KEY.

.. _clusterdb: http://www.postgresql.org/docs/current/static/app-clusterdb.html
.. _pg_reorg: http://reorg.projects.pgfoundry.org/


Examples
--------

Execute the following command to perform an online CLUSTER of all tables in
test database::

    $ pg_repack test

Execute the following command to perform an online VACUUM FULL to foo table in
test database::

    $ pg_repack --no-order --table foo -d test


Options
-------

pg_repack has the following command line options:

Reorg Options
^^^^^^^^^^^^^

Options to order rows. If not specified, pg_repack performs an online CLUSTER
using cluster indexes. Only one option can be specified. You may also specify
target tables or databases.

``-n``, ``--no-order``
    Do online VACUUM FULL.

``-o COLUMNS [,...]``, ``--order-by=COLUMNS [,...]``
    Do online CLUSTER ordered by specified columns.

``-t TABLE``, ``--table=TABLE``
    Reorganize table only. If you don't specify this option, all tables in
    specified databases are reorganized.

``-T SECS``, ``--wait-timeout=SECS``
    pg_repack needs to take an exclusive lock at the end of the
    reorganization.  This setting controls how long it wait for acquiring the
    lock in seconds. If the lock cannot be taken even after the duration,
    pg_repack forces to cancel conflicted queries. Also, if the server version
    is 8.4 or newer, pg_repack forces to disconnect conflicted backends after
    twice time passed. The default is 60 seconds.

``-Z``, ``--no-analyze``
    Disable ANALYZE after the reorganization. If not specified, run ANALYZE
    after the reorganization.

Connection Options
^^^^^^^^^^^^^^^^^^

Options to connect to servers. You cannot use ``--all`` and ``--dbname`` or
``--table`` together.

``-a``, ``--all``
    Reorganize all databases.

``-d DBNAME``, ``--dbname=DBNAME``
    Specifies the name of the database to be reorganized. If this is not
    specified and ``-a`` (or ``--all``) is not used, the database name is read
    from the environment variable PGDATABASE. If that is not set, the user
    name specified for the connection is used.

``-h HOSTNAME``, ``--host=HOSTNAME``
    Specifies the host name of the machine on which the server is running. If
    the value begins with a slash, it is used as the directory for the Unix
    domain socket.

``-p PORT``, ``--port=PORT``
    Specifies the TCP port or local Unix domain socket file extension on which
    the server is listening for connections.

``-U USERNAME``, ``--username=USERNAME``
    User name to connect as.

``-w``, ``--no-password``
    Never issue a password prompt. If the server requires password
    authentication and a password is not available by other means such as a
    .pgpass file, the connection attempt will fail. This option can be useful
    in batch jobs and scripts where no user is present to enter a password.

``-W``, ``--password``
    Force the program to prompt for a password before connecting to a
    database.

    This option is never essential, since the program will automatically
    prompt for a password if the server demands password authentication.
    However, pg_repack will waste a connection attempt finding out that the
    server wants a password. In some cases it is worth typing ``-W`` to avoid
    the extra connection attempt.


Generic Options
^^^^^^^^^^^^^^^

``-e``, ``--echo``
    Echo commands sent to server.

``-E LEVEL``, ``--elevel=LEVEL``
    Choose the output message level from ``DEBUG``, ``INFO``, ``NOTICE``,
    ``WARNING``, ``ERROR``, ``LOG``, ``FATAL``, and ``PANIC``. The default is
    ``INFO``.

``--help``
    Show usage of the program.

``--version``
    Show the version number of the program.


Environment
-----------

``PGDATABASE``, ``PGHOST``, ``PGPORT``, ``PGUSER``
    Default connection parameters

    This utility, like most other PostgreSQL utilities, also uses the
    environment variables supported by libpq (see `Environment Variables`__).

    .. __: http://www.postgresql.org/docs/current/static/libpq-envars.html


Diagnostics
-----------

Error messages are reported when pg_repack fails. The following list shows the
cause of errors.

You need to cleanup by hand after fatal errors. To cleanup, execute
``$PGHOME/share/contrib/uninstall_pg_repack.sql`` to the database where the
error occured and then execute ``$PGHOME/share/contrib/pg_repack.sql``. (Do
uninstall and reinstall.)

pg_repack: repack database "template1" ... skipped
    pg_repack is not installed in the database when ``--all`` option is
    specified.

    Do register pg_repack to the database.

ERROR: pg_repack is not installed
    pg_repack is not installed in the database specified by ``--dbname``.

    Do register pg_repack to the database.

ERROR: relation "table" has no primary key
    The target table doesn't have PRIMARY KEY.

    Define PRIMARY KEY to the table. (ALTER TABLE ADD PRIMARY KEY)

ERROR: relation "table" has no cluster key
    The target table doesn't have CLUSTER KEY.

    Define CLUSTER KEY to the table. (ALTER TABLE CLUSTER)

pg_repack: query failed: ERROR: column "col" does not exist
    The target table doesn't have columns specified by ``--order-by`` option.

    Specify existing columns.

ERROR: permission denied for schema repack
    Permission error.

    pg_repack must be executed by superusers.

pg_repack: query failed: ERROR: trigger "z_repack_trigger" for relation "tbl" already exists
    The target table already has a trigger named ``z_repack_trigger``.

    Delete or rename the trigger.

pg_repack: trigger conflicted for tbl
    The target table already has a trigger which follows by
    ``z_repack_trigger`` in alphabetical order.

    Delete or rename the trigger.


Restrictions
------------

pg_repack has the following restrictions. Be careful to avoid data
corruptions.

Temp tables
^^^^^^^^^^^

pg_repack cannot reorganize temp tables.

GiST indexes
^^^^^^^^^^^^

pg_repack cannot reorganize tables using GiST indexes.

DDL commands
^^^^^^^^^^^^

You cannot do DDL commands **except** VACUUM and ANALYZE during pg_repack. In many
cases pg_repack will fail and rollback collectly, but there are some cases
which may result in data-corruption .

TRUNCATE
    TRUNCATE is lost. Deleted rows still exist after pg_repack.

CREATE INDEX
    It causes index corruptions.

ALTER TABLE ... ADD COLUMN
    It causes lost of data. Newly added columns are initialized with NULLs.

ALTER TABLE ... ALTER COLUMN TYPE
    It causes data corruptions.

ALTER TABLE ... SET TABLESPACE
    It causes data corruptions by wrong relfilenode.


Details
-------

pg_repack creates a work table in the repack schema and sorts the rows in this
table. Then, it updates the system catalogs directly to swap the work table
and the original one.


Installations
-------------

pg_repack can be built with "make" on UNIX or Linux. pgxs build framework is
used automatically. Before building, you might need to install postgres
packages for developer (postgresql-devel, etc.) and add ``pg_config`` to your
``$PATH``. ::

    $ cd pg_repack
    $ make
    $ su
    $ make install

You can also use Microsoft Visual C++ 2010 to build the program on Windows.
There are project files in the ``msvc`` folder.

Start PostgreSQL and execute the script to register functions to your
database::

    $ pg_ctl start
    $ psql -c "CREATE EXTENSION pg_repack" -d your_database


Requirements
------------

PostgreSQL versions
    PostgreSQL 8.2, 8.3, 8.4, 9.0, 9.1, 9.2

OS
    RHEL 5.2, Windows XP SP3

Disks
    Requires free disk space twice as large as the target table(s) and
    indexes. For example, if the total size of the tables and indexes to be
    reorganized is 1GB, an additional 2GB of disk space is required.


Releases
--------

* pg_repack 1.1.8

  * Added support for PostgreSQL 9.2.
  * Added support for CREATE EXTENSION on PostgreSQL 9.1 and following.
  * Give user feedback while waiting for transactions to finish  (pg_reorg
    issue #5).
  * Bugfix: Allow running on newly promoted streaming replication slaves
    (pg_reorg issue #1).
  * Bugfix: Properly escape column names (pg_reorg issue #6).
  * Bugfix: Avoid recreating invalid indexes, or choosing them as key
    (pg_reorg issue #9).

* pg_reorg 1.1.7 (2011-08-07)

  * Bugfix: VIEWs and FUNCTIONs could be corrupted that used a reorganized
    table which has a dropped column.
  * Supports PostgreSQL 9.1 and 9.2dev. (but EXTENSION is not yet)


See Also
--------

    * `clusterdb <http://www.postgresql.org/docs/current/static/app-clusterdb.html>`__
    * `vacuumdb <http://www.postgresql.org/docs/current/static/app-vacuumdb.html>`__
