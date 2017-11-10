The UI is up, but I don't see anything
======================================

Here are the things to check and fix:

Are there perfdata generated?
-----------------------------

On the Nagios server, check if perfdata are generated.  The location is defined
in the ``proces-service-perfdata-file`` and ``proces-host-perfdata-file``
commands on Nagios, as seen in :ref:`nagios_and_nagios_dispatcher` section.

Assuming default configuration (``/var/lib/nagios3/spool/perfdata/``), you can
check like this::

    # ls -al /var/lib/nagios3/spool/perfdata/

.. warning::

  Obviously, you need to have configured Nagios so that checks are actually
  performed to expect perfdata generated.

If files are getting created, you can skip to the next item. Otherwise, you need to
configure Nagios so perfdata are processed, as documented in
:ref:`nagios_and_nagios_dispatcher`. Also be careful, if the ``nagios_dispatcher``
is running, perfdata files won't stay long in the perfdata directory.

Is nagios_dispatcher running?
-----------------------------

If perfdata are being generated but are accumulating, you have an issue with
the :ref:`nagios dispatcher <nagios_dispatcher>`:

* check and double check the configuration file

    * perfdata directory
    * connection credentials
    * host and port

* check the connection with the opm database

    * PostgreSQL logs
    * pg_hba and/or credentials

* check that the daemon is running::

    root:~# ps aux | grep nagios_dispatcher

* check the nagios dispatcher logs

* beware: ``debug=1`` in the dispatcher configuration file will make it stop
at the first problem in the data.

When the perfdata are being removed from the perfdata directory, you can move
to the next item.

Are the perfdata accumulating in the hub table?
-----------------------------------------------

If the nagios_dispatcher is running and perfdata files being removed, lines
should be added in the ``wh_nagios.hub`` table of the ``opm`` database::

  opm=# SELECT COUNT(*) FROM wh_nagios.hub;

.. warning::

  If you don't see any line appearing in this table, it's very likely that
  you're looking at the wrong server and/or the wrong database.

According to the :ref:`wh_nagios` documentation, you should have setup a cron
to call the ``wh_nagios.dispatch_record()`` stored function every minute.

If after some minutes the number of records doesn't fall, there's a problem with
this cron. Make sure to double check::

* output of this cron if you redirected it to a file or a mail
* credential access (you can use a `.pgpass file
  <http://www.postgresql.org/docs/current/static/libpq-pgpass.html>`_ if needed)
* host, port, user, database...
* check the connection with the opm database

    * PostgreSQL logs
    * pg_hba and/or credentials

Once the ``wh_nagios.dispatch_record()`` is successfully called, the data will
appear in the user tables. You can check for instance the number of servers the
UI knows about::

    opm=# SELECT COUNT(*) FROM public.servers;

I still can't see anything in the UI
------------------------------------

* check the credentials for the :ref:`dedicated UI user<ui_configuration>`
* check the :ref:`UI configuration file<ui_configuration>`

* check the connection with the opm database

    * PostgreSQL logs
    * pg_hba and/or credentials

* check the UI logs.  For instance, if you used an
  :ref:`Apache server<ui_apache>`, the ``opm.log``.  If you tried with the
  :ref:`morbo tool<ui_morbo>`, then the standard output.
* check that you connect to the good OPM UI server.

