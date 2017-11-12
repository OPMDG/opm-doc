Installation
============

Requirements
------------
To install OPM, you need a 9.3 or more PostgreSQL cluster, standard compiling tools and Nagios. The PostgreSQL cluster and Nagios can be installed on the servers you want, and can be installed on the same server.

System
------

Install the PostgreSQL development packages of your Linux distribution if necessary (e.g. ``postgresql96-devel`` on Red Hat, ``postgresql-server-dev-9.6`` on Debian).

The tool ``pg_config`` is required. Distributions packages put it usually in ``/usr/pgsql-9.6/bin/pg_config`` (Red Hat) or ``/usr/lib/postgresql/9.6/bin/pg_config`` (Debian). It may be necessary to put it in your ``$PATH`` at least temporarily (e.g. ``export PATH=/usr/lib/postgresql/9.6/bin:$PATH``), especially if you have more than one version of PostgreSQL on the server.

We suppose that the repositories **opm-core** (https://github.com/OPMDG/opm-core) and **opm-wh_nagios** (https://github.com/OPMDG/opm-wh_nagios) are stored into **/usr/local/src/opm/** (your OPM directory).

OPM core
--------

We need to install the core of OPM first. From your opm directory, as user ``root``::

    root:/usr/local/src/opm# cd opm-core/pg
    root:/usr/local/src/opm/opm-core/pg# make install

It will copy some files into the extensions directory of PostgreSQL.

Then, using a superuser role:

.. code-block:: sql

    postgres@postgres=# CREATE DATABASE opm;
    postgres@postgres=# \c opm
    postgres@opm=# CREATE EXTENSION opm_core;


You'll need to create a first OPM administrator account:

.. code-block:: sql

    postgres@opm=# SELECT create_admin('admin1', 'agoodpassword');

**This is the user you'll need to log on the UI.**

.. _wh_nagios:

wh_nagios
---------

To install the module "wh_nagios", from your OPM directory as user "root"::

    root:/usr/local/src/opm# cd opm-wh_nagios/pg
    root:/usr/local/src/opm/wh_nagios/pg# make install


Then, using a superuser role, in your **opm** database:

.. code-block:: sql

    postgres@opm=# CREATE EXTENSION hstore;
    CREATE EXTENSION

    postgres@opm=# CREATE EXTENSION wh_nagios;
    CREATE EXTENSION


Then, you need to create a crontab that will process incoming data and dispatch them. For instance, to trigger it every minute::

    * * * * * psql -c 'SELECT wh_nagios.dispatch_record()' opm

This crontab can belong to any user, as long as it can connect to the PostgreSQL
``opm`` database with any PostgreSQL role.

To import data into a warehouse, you need a PostgreSQL role. We recommand to
create a dedicated role, for instance:

.. code-block:: sql

    postgres@opm=# CREATE ROLE opm_dispatcher LOGIN PASSWORD 'anothergoodpassword';

You must then allow this role to import data in a warehouse, by calling
``public.grant_dispatch``. For instance, if the PostgreSQL role is ``opm`` and
the warehouse is ``wh_nagios``::

    postgres@opm=# SELECT grant_dispatcher('wh_nagios', 'opm_dispatcher');

.. _nagios_and_nagios_dispatcher:

Nagios & nagios_dispatcher
--------------------------

The dispatcher ``nagios_dispatcher.pl`` aims to dispatch perfdata from Nagios files to the ``wh_nagios`` warehouse.

``nagios_dispatcher`` requires the DBD::Pg perl module. It is packaged in ``perl-DBD-Pg`` (Red Hat), ``libdbd-pg-perl`` (Debian) or in the CPAN.

We'll need first to setup Nagios to create its perfdata files that "nagios_dispatcher" will poll and consume. As ``root``, create the command file and destination folder::


    root:~# mkdir -p /var/lib/nagios3/spool/perfdata/
    root:~# chown nagios: /var/lib/nagios3/spool/perfdata/
    root:~# cat <<'EOF' >> /etc/nagios3/commands.cfg
    define command{
        command_name    process-service-perfdata-file
        command_line    /bin/mv /var/lib/nagios3/service-perfdata /var/lib/nagios3/spool/perfdata/service-perfdata.$TIMET$
    }
    define command{
        command_name    process-host-perfdata-file
        command_line    /bin/mv /var/lib/nagios3/host-perfdata /var/lib/nagios3/spool/perfdata/host-perfdata.$TIMET$
    }
    EOF

Then, in your Nagios main configuration file, make sure the following parameter are set accordingly::

    process_performance_data=1
    host_perfdata_file=/var/lib/nagios3/host-perfdata
    service_perfdata_file=/var/lib/nagios3/service-perfdata
    host_perfdata_file_processing_command=process-host-perfdata-file
    service_perfdata_file_processing_command=process-service-perfdata-file
    host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tHOSTOUTPUT::$HOSTOUTPUT$
    service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$\tSERVICEOUTPUT::$SERVICEOUTPUT$
    host_perfdata_file_mode=a
    service_perfdata_file_mode=a
    host_perfdata_file_processing_interval=15
    service_perfdata_file_processing_interval=15

.. note::

    If you're using Icinga2 instead of Nagios, you need instead to:

    * enable perfdata::

        $ icinga2 feature enable perfdata

    * configure data format in ``/etc/icinga2/features-enabled/perfdata.conf``::

        library "perfdata"
        object PerfdataWriter "perfdata" {
            host_perfdata_path = "/var/spool/icinga2/perfdata/host-perfdata"
            service_perfdata_path = "/var/spool/icinga2/perfdata/service-perfdata"
            rotation_interval = 15s
            host_format_template = "DATATYPE::HOSTPERFDATA\tTIMET::$icinga.timet$\tHOSTNAME::$host.name$\tHOSTPERFDATA::$host.perfdata$\tHOSTCHECKCOMMAND::$host.check_command$\tHOSTSTATE::$host.state$\tHOSTSTATETYPE::$host.state_type$\tHOSTOUTPUT::$host.output$"
            service_format_template = "DATATYPE::SERVICEPERFDATA\tTIMET::$icinga.timet$\tHOSTNAME::$host.name$\tSERVICEDESC::$service.name$\tSERVICEPERFDATA::$service.perfdata$\tSERVICECHECKCOMMAND::$service.check_command$\tHOSTSTATE::$host.state$\tHOSTSTATETYPE::$host.state_type$\tSERVICESTATE::$service.state$\tSERVICESTATETYPE::$service.state_type$\tSERVICEOUTPUT::$service.output$"
        }

    Icinga2 has different macros names from Nagios. For a complete list see
    `documentation <http://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-basics#host-runtime-macros>`_.


.. note::

    Beware: the perfdata files can accumulate very quickly if not consumed by the ``nagios_dispatcher`` script.

.. _nagios_dispatcher:

Create the dispatcher configuration file::

    root:~# mkdir -p /usr/local/etc/
    root:~# cat <<EOF > /usr/local/etc/nagios_dispatcher.conf
    daemon=1
    directory=/var/lib/nagios3/spool/perfdata/
    frequency=5
    db_connection_string=dbi:Pg:dbname=opm host=127.0.0.1
    db_user=YOUR_USER
    db_password=YOUR_PASS
    debug=0
    syslog=1
    hostname_filter = /^$/ # Empty hostname. Never happens
    service_filter = /^$/ # Empty service
    label_filter = /^$/ # Empty label
    EOF

    root:~# chown nagios /usr/local/etc/nagios_dispatcher.conf

.. note::

    With our previous examples, **db_user** would've been set to ``opm_dispatcher`` and
    **db_password** should be set to ``anothergoodpassword``. If you use Icinga2,
    ``directory`` must be set to ``/var/spool/icinga2/perfdata/`` (Icinga2 default
	directory for perfdata).

Install the ``nagios_dispatcher.pl`` file into ``/usr/local/bin/`` ::

    root:~# cp /usr/local/src/opm/wh_nagios/bin/nagios_dispatcher.pl /usr/local/bin

You can test that it works using the command line (you may have to set ``daemon=0``)::

    /usr/local/bin/nagios_dispatcher.pl --verbose -c /usr/local/etc/nagios_dispatcher.conf

The files in the Nagios ``perfdata`` directory must disappear one by one.


**If your operating system uses systemd**

In ``nagios_dispatcher.conf`` you must set ``daemon`` to 0 and modify the connection string.
The full file becomes::

    daemon=0
    directory=/var/lib/nagios3/spool/perfdata/
    frequency=5
    db_connection_string=dbi:Pg:dbname=opm;host=127.0.0.1
    db_user=YOUR_USER
    db_password=YOUR_PASS
    debug=0
    syslog=1
    hostname_filter = /^$/ # Empty hostname. Never happens
    service_filter = /^$/ # Empty service
    label_filter = /^$/ # Empty label

Create the file ``/etc/systemd/system/nagios_dispatcher.service`` with the following content::

    [Unit]
    Description=Nagios Dispatcher Service
    After=network.target

    [Service]
    Type=simple
    User=nagios
    ExecStart=/usr/local/bin/nagios_dispatcher.pl -c /usr/local/etc/nagios_dispatcher.conf
    Restart=on-abort

    [Install]
    WantedBy=multi-user.target

Now enable and start the service::

    systemctl enable nagios_dispatcher
    systemctl start nagios_dispatcher

Check that the ``nagios_dispatcher`` process shows up in the process list.

**If your operating system uses inittab**

Add the following line at the end of the ``/etc/inittab`` file::

    d1:23:respawn:/usr/bin/perl -w /usr/local/bin/nagios_dispatcher.pl --daemon --config /usr/local/etc/nagios_dispatcher.conf

and reload it::

    root:~# init q

**If your operating system uses upstart**

Create the file ``/etc/init/nagios_dispatcher.conf``, with the following content::

    # This service maintains nagios_dispatcher

    start on stopped rc RUNLEVEL=[2345]
    stop on starting runlevel [016]

    respawn
    exec /usr/local/bin/nagios_dispatcher.pl -c /usr/local/etc/nagios_dispatcher.conf

and start the job::

    root:~# initctl start nagios_dispatcher

User interface
--------------

The default user interface is based on the web framework Mojolicious_. You need to install:

* Perl (5.10 or above)
* Mojolicious (4.63 or above, **less than 5.0 !**)
* Mojolicious::Plugin::I18N (version 0.9)
* DBD::Pg perl module
* PostgreSQL (9.3 or above)
* A CGI/Perl webserver

You can install "Mojolicious" with your Linux distribution package system if old enough
packages of Mojolicious are available.

Another option is from CPAN::

    curl -L cpanmin.us | perl - Mojolicious@4.99
    curl -L cpanmin.us | perl - Mojolicious::Plugin::I18N@0.9
    curl -L cpanmin.us | perl - DBI
    curl -L cpanmin.us | perl - DBD::Pg

Alternatively, you can download the required archives and install them manually::

    wget http://backpan.perl.org/authors/id/S/SR/SRI/Mojolicious-4.99.tar.gz
    tar xzf Mojolicious-4.99.tar.gz
    cd Mojolicious-4.99
    perl Makefile.PL
    make
    sudo make install
    cd ..
    wget http://backpan.perl.org/authors/id/S/SH/SHARIFULN/Mojolicious-Plugin-I18N-0.9.tar.gz
    tar xzf Mojolicious-Plugin-I18N-0.9.tar.gz
    cd Mojolicious-Plugin-I18N-0.9
    make
    sudo make install

To install the UI plugin ``wh_nagios`` (or any other UI plugin), from your ``opm`` directory as user ``root``::

    root:/usr/local/src/opm# cd opm-core/ui/modules
    root:/usr/local/src/opm/opm-core/ui/modules# ln -s /usr/local/src/opm/opm-wh_nagios/ui wh_nagios

.. _ui_opmuser:

Then, on your OPM database side, you need to create another user for the UI::

    postgres@opm=# CREATE USER opmui WITH ENCRYPTED PASSWORD 'yetanothergoodpassword';
    postgres@opm=# SELECT * from grant_appli('opmui');


.. _ui_configuration:

Finally, in ``/usr/local/src/opm/opm-core/ui``, copy the **opm.conf-dist** file to **opm.conf**, and edit it to suit you needs, for instance::

    {
        ...
        "database" : {
            "dbname"   : "opm",
            "host"     : "127.0.0.1",
            "port"     : "5432",
            "user"     : "opmui",
            "password" : "opmui"
        },
        ...
        "plugins" : [ "wh_nagios" ]
    }

**This user is only needed for the connection between the UI and the database. You only have to use it in the opm.conf file**

.. _ui_morbo:

To test the web user interface quickly, you can use either ``morbo`` or ``hypnotoad``, both installed with Mojolicious. Example with Morbo::

    user:/usr/local/src/opm/opm-core/ui/opm$ morbo script/opm
    [Fri Nov 29 12:12:52 2013] [debug] Helper "url_for" already exists, replacing.
    [Fri Nov 29 12:12:52 2013] [debug] Reading config file "/home/ioguix/git/opm/ui/opm/opm.conf".
    [Fri Nov 29 12:12:53 2013] [info] Listening at "http://*:3000".
    Server available at http://127.0.0.1:3000.

* Alternativeley, this example uses ``hypnotoad``, which suits production better::

    user:/usr/local/src/opm-core/ui/opm$ hypnotoad -f script/opm

.. note::

    Removing "-f" makes it daemonize.

* To configure nginx to forward requests to a ``hypnotoad`` application server::

    upstream hypnotoad {
      server 127.0.0.1:8080;
    }

    server {
      listen 80;

      location / {
            proxy_pass http://hypnotoad;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto "http";
      }
    }

.. note::

  You should ensure that hypnotoad starts on boot, e.g. in **/etc/rc.local**::

  .. code-block:: bash

    su - www-data -c 'hypnotoad /var/www/opm-core/ui/script/opm'

.. _ui_apache:

If you want to use Apache, here is a quick configuration sample using CGI::

        <VirtualHost *:80>
                ServerAdmin webmaster@example.com
                ServerName opm.example.com
                DocumentRoot /var/www/opm/public/

                <Directory /var/www/opm/public/>
                        AllowOverride None
                        Order allow,deny
                        allow from all
                        IndexIgnore *

                        RewriteEngine On
                        RewriteBase /
                        RewriteRule ^$ opm.cgi [L]
                        RewriteCond %{REQUEST_FILENAME} !-f
                        RewriteCond %{REQUEST_FILENAME} !-d
                        RewriteRule ^(.*)$ opm.cgi/$1 [L]
                </Directory>

                ScriptAlias /opm.cgi /var/www/opm/script/opm
                <Directory /var/www/opm/script/>
                        AddHandler cgi-script .cgi
                        Options +ExecCGI
                        AllowOverride None
                        Order allow,deny
                        allow from all
                        SetEnv MOJO_MODE production
                        SetEnv MOJO_MAX_MESSAGE_SIZE 4294967296
                </Directory>

                ErrorLog ${APACHE_LOG_DIR}/opm.log
                # Possible values include: debug, info, notice, warn, error, crit,
                # alert, emerg.
                LogLevel warn

                CustomLog ${APACHE_LOG_DIR}/opm.log combined
        </VirtualHost>

(assuming that the directory **/usr/local/src/opm/opm-core/ui** has been symlinked to **/var/www/opm**).

For a complete list and specifications on supported http servers, please check the `Mojolicious official documentation
<http://mojolicio.us/perldoc/Mojolicious/Guides/Cookbook#DEPLOYMENT>`_.

.. _Mojolicious: http://www.mojolicio.us/
