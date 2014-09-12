FAQ
===

Do we really need yet another PostgreSQL monitoring tool?
---------------------------------------------------------

We do think so. We are aware that there's a lot of similar tools ( Postgres Enterprise Manager, pgObserver or pgAnalyzer,  just to name a few). However we feel that most them are not entirely open-source and/or they are owned by a single company. 

We aim to create a free alternative to Oracle Enterprise Manager and we want to build this tool with the help and contributions of the PostgreSQL community.


I don't how to use Nagios / I don't want to use Nagios. Is there any hope?
--------------------------------------------------------------------------

Yes ! Currently our data collection process is based on Nagios, because it's one of the most widespread monitoring engine. So if you want to use the current version, you need to install Nagios.

However we build OPM as an agnostic system that can be plugged on any scheduler. Therefore it is possible de developped connectors with tools like zabbix, centreon, shinken and others. We plan to do it eventually but we don't have a clear roadmap for it yet. If you're interested by such connectors, let us know !

I need enterprise-grade support for this software
-------------------------------------------------

DALIBO, as the main sponsor of the project, can provide training and support for both PostgreSQL and OPM. See http://www.dalibo.com for more details.
