FAQ
===


Do you have any company using this tool? How many if you know?
--------------------------------------------------------------------

OPM is an open source software, which means we don't have a extensive
list of companies using it. 

However DALIBO (the main sponsor for this project) has dozens of customers using it. And we do know about an OPM server monitoring more than 100
PostgreSQL instances as we speak.


Do you need agents to install on the remote servers? 
-----------------------------------------------------------

To monitor a PostgreSQL instance, you need to install an agent on this
server. For now, there's only one agent available which is called
**check_pgactivity**. 

More details here : https://github.com/OPMDG/check_pgactivity

It's a Nagios agent but it should work fine with
Nagios-compatible software (see below)


Is possible to use the check_pgactivity agent with Nagios-compatible tools such as  Icinga / Shinken /  Naemon / ....  ?
----------------------------------------------------------------------------

We know that the Nagios project is currently in a bad shape. But it's an industry standard. There's some very interesting Nagios forks nowadays and as long as they maintain backward compatibilty with Nagios, we should be able to use the check_pgactivity agent with them. We cannot test our agent on every Nagios-compatible software so if you do use it with Icinga, Shinken or Naemon please let us know !


I don't want to use Nagios or a Nagios compatible software. Is there any hope?
--------------------------------------------------------------------------

Yes ! Currently our data collection process is based on Nagios, because it's one of the most widespread monitoring engine. So if you want to use the current version, you need to install Nagios.

However we build OPM as an agnostic system that can be plugged on any scheduler. Therefore it is possible de developped connectors with tools like zabbix, centreon, shinken and others. We plan to do it eventually but we don't have a clear roadmap for it yet. If you're interested by such connectors, let us know !


What is the performance impact of this tool on a postgreSQL production database?
----------------------------------------------------------------------------------


The performance impact is hard to define precisely. It will really depend
on the frequency of the stats collection processes. You can configure
Nagios to get stats every 30s or every 5 min: the impact on performance
will be very different. So basically the performance impact is more a
question of how you configure Nagios and doesn't really depend on OPM itself.




Do we really need yet another PostgreSQL monitoring tool?
---------------------------------------------------------

We do think so. We are aware that there's a lot of similar tools ( Postgres Enterprise Manager, pgObserver or pgAnalyzer,  just to name a few). However we feel that most them are not entirely open-source and/or they are owned by a single company.

We aim to create a free alternative to Oracle Enterprise Manager and we want to build this tool with the help and contributions of the PostgreSQL community.


I need enterprise-grade support for this software
-------------------------------------------------

DALIBO, as the main sponsor of the project, can provide training and support for both PostgreSQL and OPM. See http://www.dalibo.com for more details.
