.. _ussuri-priorities:

=========================
Ussuri Project Priorities
=========================

List of priorities the Monasca drivers team is prioritizing in Ussuri.

The owners listed are responsible for tracking the status of that work and
helping get that work done. They are not the only contributors to this work,
and not necessarily doing most of the coding!

The implementation progress on these priorities and other identified important
tasks is tracked in `this board`_.

.. _this board: https://storyboard.openstack.org/#!/board/190

Essential Priorities
~~~~~~~~~~~~~~~~~~~~

+-------------------------------------------------+---------------------------+
| Title                                           | Owners                    |
+=================================================+===========================+
| `New thresholding engine`_                      | chaconpiza                |
+-------------------------------------------------+---------------------------+
| `Monasca Events Agent`_                         | witek                     |
+-------------------------------------------------+---------------------------+
| `IPv6 support`_                                 | witek                     |
+-------------------------------------------------+---------------------------+


High Priorities
~~~~~~~~~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| `InfluxDB HA Setup`_                        | dougsz                  |
+---------------------------------------------+-------------------------+
| `Query Logs API`_                           | dougsz                  |
+---------------------------------------------+-------------------------+
| `Application Credentials`_                  | dougsz                  |
+---------------------------------------------+-------------------------+
| `New InfluxDB query capabilities`_          | dougsz                  |
+---------------------------------------------+-------------------------+
| `Middleware upgrade`_                       | witek                   |
+---------------------------------------------+-------------------------+

Optional Priorities
~~~~~~~~~~~~~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| Sharding model for InfluxDB                 |                         |
+---------------------------------------------+-------------------------+
| Refresh Monasca transform engine            | joadavis                |
+---------------------------------------------+-------------------------+

Details
~~~~~~~

New thresholding engine
--------------------------------------------

`Faust library`_ has been evaluated and the prototype of the thresholding
engine based on this library has been implemented. The goal of this effort is
to implement the new thresholding engine for Monasca to replace Apache Storm
Java application.

.. _Faust library: https://faust.readthedocs.io

Monasca Events Agent
--------------------

The goal is to implement Monasca Events Listener which will publish Openstack
notifications and events from third party applications to Monasca Events API.
`Specification`_ listing existing requirements and proposed implementation
has been written up in the past.

.. _Specification: http://specs.openstack.org/openstack/monasca-specs/specs/stein/approved/monasca-events-listener.html

IPv6 Support
------------

It is the community wide goal to `support IPv6-Only Deployments`_.

.. _support IPv6-Only Deployments: https://governance.openstack.org/tc/goals/selected/train/ipv6-support-and-testing.html

InfluxDB HA Setup
-----------------

Story: https://storyboard.openstack.org/#!/story/2005620

Query Logs API
--------------

`Add support`_ for querying ElasticSearch via the Monasca Log API to support tenant
scoped access to logs.

.. _Add support: https://blueprints.launchpad.net/monasca/+spec/log-query-api

Application Credentials
-----------------------

`Keystone appliction credentials <https://docs.openstack
.org/keystone/latest/user/application_credentials.html>`_ offer the mechanism
to allow applications to authenticate to Keystone. The ability to specify
`access rules <http://specs.openstack
.org/openstack/keystone-specs/specs/keystone/stein/capabilities-app-creds
.html>`_ for application credentials has been implemented in the Train cycle.

The goal of this story is to add application credentials support in
*monasca-agent*. This will prevent the security risk of revealing OpenStack
user's password when installing the agent on the tenants environment. The
access rules of these application credentials should be limited to posting
measurements. *monasca-setup* should be extended to automatically generate such
credentials and save them in configuration file if needed.

Similar task should be implemented in *monasca-grafana-datasource*.

Stories:

* https://storyboard.openstack.org/#!/story/2005622
* https://storyboard.openstack.org/#!/story/2005623

New InfluxDB Query Capabilities
-------------------------------

The goal is to extend the Monasca API to query measurements using aggregation
functions available in InfluxDB, like e.g. DERIVATIVE(). Another goal is to
investigate the new Flux QL to allow basic arithmetic operations between
different measurements, e.g. (disk_used / disk_total).

Middleware upgrade
------------------

Story: https://storyboard.openstack.org/#!/story/2006768

