Warehouses
==========

Overview
--------

A warehouse is used to store a kind of data. In version 1 and 2, the only available warehouse is wh_nagios, which stores Nagios' perfdata.

Each warehouse must have a unique name, lowercase, with a leading **wh_**, and it's own schema, named as the warehouse (with the leading **wh_**). All objects have to be in this schema, and should probably be configured to be dumped.

Content
-------

A warehouse should at least provide a **pg** subdirectory containing a PostgreSQL extension depending on **opm-core** extension. It can also provide an **ui** subdirectory if the warehouse wants to provide some ui content.
Then, it can also provides various subdirectories for its need. For instance, *wh_nagios* warehouse provides a **bin** subdirectory containing the *nagios_dispatcher* tool.

Therefore, a typical warehouse structure would be::

    wh_my_warehouse
     \_ pg
     \_ ui

Implementing the PostgreSQL extension
-------------------------------------

ACL don't have to be handled by the warehouse, as the only regular database access should be done by the ui. The ACL are handled by the opm_core extension. Only a few tables and stored functions have to be implemented (see below).

-------

In order to integrate with the opm_core module, the warehouse extension has to implement at least some objects.

Tables
^^^^^^

* **services**:

A table that inherits public.services and its constraints, which will store every needed information for a service within the warehouse.
A typical declaration will look like::

    CREATE TABLE wh_name.services (
      useful_col     datatype,
      ...
      PRIMARY KEY (id),
      FOREIGN KEY (id_server) REFERENCES public.servers (id) ON UPDATE CASCADE ON DELETE CASCADE),
      UNIQUE (id_server, service)
    ) INHERITS (public.services);
    SELECT pg_catalog.pg_extension_config_dump('wh_name.services', '') ;

* **metrics**:

A table that inherits public.metrics and its constraints, which will store every metrics (label information on all graphs) for every service within the warehouse.
A typical declaration will look like::

    CREATE TABLE wh_name.metrics (
        useful_col         datatype,
        ...
        PRIMARY KEY (id),
        FOREIGN KEY (id_service) REFERENCES wh_name.services (id) MATCH FULL ON DELETE CASCADE ON UPDATE CASCADE
    )
    INHERITS (public.metrics);
    SELECT pg_catalog.pg_extension_config_dump('wh_name.metrics', '') ;


* **series**:

A table that inherits public.series and its constraints, which will store association between metrics and graphs for every service within the warehouse.
A typical declaration will look like::

    CREATE TABLE wh_nagios.series (
        FOREIGN KEY (id_graph)  REFERENCES public.graphs (id) MATCH FULL ON DELETE CASCADE ON UPDATE CASCADE,
        FOREIGN KEY (id_metric) REFERENCES wh_name.metrics (id) MATCH FULL ON DELETE CASCADE ON UPDATE CASCADE
    )
    INHERITS (public.series);
    CREATE UNIQUE INDEX ON wh_name.series (id_metric, id_graph);
    CREATE INDEX ON wh_name.series (id_graph);
    SELECT pg_catalog.pg_extension_config_dump('wh_name.series', '') ;


Stored functions
^^^^^^^^^^^^^^^^

* **get_metric_data((id_metric bigint, timet_begin timestamp with time zone, timet_end timestamp with time zone) RETURNS TABLE (timet timestamp with time zone, value numeric)**:

Function that will be called by opm_core when displaying a graph. It should return all timestamped stored value for a specific metric within the specified interval. The data don't need to be ordered by the timestamp.

* **grant_dispatcher(p_rolname text) RETURNS TABLE (operat text, approle name, appright text, objtype text, objname text)**:

Function that will be called by opm_core, when granting the right a role to dispatch data. Is must return the list of all objects granted. This is meant to grant CONNECT, USAGE, INSERT... permission on the warehouse's objects that store data.

* **revoke_dispatcher(p_rolname text) RETURNS TABLE (operat text, approle name, appright text, objtype text, objname text)**:

Function that will be called by opm_core, when revoking from a role to dispatch data. This function is the exact opposite of **grant_dispatcher**, GRANT being replaced with REVOKE.

* **purge_service(VARIADIC bigint[]) RETURNS bigint**:

Function that will purge data according to the related **servalid** interval. It must return the number of services actually purged.

------

And optionally:

* **cleanup_service(id_service bigint)**:

This function won't be called by the core module. Each warehouse has to handle his way of cleaning data (if needed). It has to update the warehouse's **services.last_cleanup** column when executed.

