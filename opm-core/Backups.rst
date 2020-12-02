Backups & Restore
=================

Backup
------

To create logical backups of the OPM database, use pg_dump as usual.

Restoring logical backups
-------------------------

In a general way, restoring a backup created with pg_dump should not be a problem, as the backup contains the necessary `CREATE EXTENSION` orders. The extensions must be installed for the target PostgreSQL version (see `Installation <Installation>`).

You may wish to parallelize the restoration to spare time. In this case, it may fail with a foreign key error, as the extensions install tables with foreign keys at the very beginning of the restore, and pg_restore does not try to respect this order. The trick is to parallelize only the restoration of the biggest tables.

The following commands assume that the database and the roles were already created :

.. code-block:: bash

    pg_restore -e --section=pre-data -d opm  opm.dump
    pg_restore -e --section=data -l opm.dump|grep -v 'service_counters_' > restore.1.list
    pg_restore -e --section=data -l opm.dump|grep  'service_counters_' > restore.2.list
    pg_restore -e -d opm  -L restore.1.list opm.dump
    pg_restore -e -d opm  -L restore.2.list opm.dump  --jobs=4
    pg_restore -e --section=post-data -d opm  opm.dump --jobs=4
