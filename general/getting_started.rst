Getting started
===============

Overview
--------

OPM is a set of tools working above Nagios (or similar)
to store into a database the perfdata retrieved by any Nagios agent
and graph them in a web dashboard.

check_pgactivity (https://github.com/OPMDG/check_pgactivity) is a Nagios agent
dedicated to PostgreSQL.
It is technically separated from OPM.


Requirements & Installation
---------------------------

We suppose that you have in place and configured to your need:
  * a working Nagios (or similar) installation
  * agents displaying perfdata (e.g. check_pgactivity or other Nagios probes)
  * an empty PostgreSQL database (9.3 or later) on the instance of your choice
    to store the data.

See the :doc:`Installation <../opm-core/Installation>` page to install:
  * the :ref:`opm_core <opm_core>` and :ref:`wh_nagios <wh_nagios>` extensions
  * the :ref:`nagios dispatcher script<nagios_and_nagios_dispatcher>` to
    retrieve the perfdata
  * the :ref:`web user interface <user_interface>`.
