Installation
============

Requirements
------------
To install OPM, you need a 9.3 or more PostgreSQL cluster, standard compiling tools and Nagios. The PostgreSQL cluster and Nagios can be installed on the servers you want, and can be installed on the same server.

System
------

The tool "pg_config" is required, install the PostgreSQL development packages of your Linux distribution if necessary. We suppose that the repositories **opm-core** and **opm-wh_nagios** are stored into **/usr/local/src/opm/**.

OPM core
--------

We need to install the core of opm first. From your opm directory as user "root"::

    root:/usr/local/src/opm# cd opm-core/pg
    root:/usr/local/src/opm/opm-core/pg# make install


Then, using a superuser role::

    postgres@postgres=# CREATE DATABASE opm;
    postgres@postgres=# \c opm
    postgres@opm=# CREATE EXTENSION opm_core;


You'll need to create a first opm admin account::

    postgres@opm=# SELECT create_admin('admin1', 'admin1');

**This is the user you'll need to log on the UI**


wh_nagios
---------

To install the module "wh_nagios", from your opm directory as user "root"::

    root:/usr/local/src/opm# cd opm-wh_nagios/pg
    root:/usr/local/src/opm/wh_nagios/pg# make install


Then, using a superuser role::

    postgres@opm=# CREATE EXTENSION hstore;
    CREATE EXTENSION

    postgres@opm=# CREATE EXTENSION wh_nagios;
    CREATE EXTENSION


Then, you need to create a crontab that will process incoming data and dispatch them. As instance, to trigger it every minute::

    * * * * * psql -c 'SELECT wh_nagios.dispatch_record()' opm

This crontab can belong to any user, as long as it can connect to the PostgreSQL opm database.

To allows a opm role to import data in a warehouse, you need to call "public.grant_dispatch". As instance, for a opm role called "user1" on warehouse "wh_nagios"::

    postgres@opm=# SELECT grant_dispatcher('wh_nagios', 'user1');


Nagios & nagios_dispatcher
--------------------------

The dispatcher "nagios_dispatcher" aimed to dispatch perfdata from Nagios files to the "wh_nagios" warehouse.

"nagios_dispatcher" require the DBD::Pg perl module. Make sure to install it on your system . Eg. under debian::

    root:~# apt-get install libdbd-pg-perl


We'll need first to setup Nagios to create its perdata files that "nagios_dispatcher" will poll and consume. As user "root", create to required command file and destination folder::


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


The dispatcher itself::

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

Install the nagios_dispatcher.pl file into the /usr/local/bin/ directory::

    root:~# cp /usr/local/src/opm/ /usr/local/bin

Add the following line at the end of the /etc/inittab file::

    d1:23:respawn:/usr/bin/perl -w /usr/local/bin/nagios_dispatcher.pl --daemon --config /usr/local/etc/nagios_dispatcher.conf

Reload the /etc/inittab file::

    root:~# init q


User interface
--------------

The default user interface is based on the web framework Mojolicious_. You need to install:

* Perl (5.10 or above)
* Mojolicious (4.63 or above, less than 5.0)
* Mojolicious::Plugin::I18N (version 0.9)
* DBD::Pg perl module
* PostgreSQL (9.3 or above)
* A CGI/Perl webserver

You can install "Mojolicious" using CPAN or your Linux distribution package system if available. Here is an example with CPAN::

    curl -L cpanmin.us | perl - Mojolicious@4.99
    curl -L cpanmin.us | perl - Mojolicious::Plugin::I18N@0.9

To install the UI plugin "wh_nagios" (or any other UI plugin), from your opm directory as user "root"::

    root:/usr/local/src/opm# cd opm-core/ui/modules
    root:/usr/local/src/opm/opm-core/ui/modules# ln -s /usr/local/src/opm/opm-wh_nagios/ui wh_nagios

Then, on your OPM database side, you need to create an opm user for the UI::

    postgres@opm=# CREATE USER opmui WITH ENCRYPTED PASSWORD 'opmui';
    postgres@opm=# SELECT * from grant_appli('opmui');


Finally, in the directory **/usr/local/src/opm/opm-core/ui**, copy the **opm.conf-dist** file to **opm.conf**, and edit it to suit you needs, for instance::

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


To test the web user interface quickly, you can use either "morbo" or "hypnotoad", both installed with Mojolicious. Example with Morbo::

    user:/usr/local/src/opm/opm-core/ui/opm$ morbo script/opm
    [Fri Nov 29 12:12:52 2013] [debug] Helper "url_for" already exists, replacing.
    [Fri Nov 29 12:12:52 2013] [debug] Reading config file "/home/ioguix/git/opm/ui/opm/opm.conf".
    [Fri Nov 29 12:12:53 2013] [info] Listening at "http://*:3000".
    Server available at http://127.0.0.1:3000.

Using "hypnotoad", which suit better for production::

    user:/usr/local/src/opm/ui/opm/opm-core$ hypnotoad -f script/opm

Removing "-f" makes it daemonize.

If you want to use "apache", here is a quick configuration sample using CGI::

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
