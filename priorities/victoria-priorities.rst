.. _victoria-priorities:

===========================
Victoria Project Priorities
===========================

List of priorities the Monasca drivers team is prioritizing in Victoria.

The owners listed are responsible for tracking the status of that work and
helping get that work done. They are not the only contributors to this work,
and not necessarily doing most of the coding!

The implementation progress on these priorities and other identified important
tasks is tracked in `this board`_.

.. _this board: https://storyboard.openstack.org/#!/board/217

Essential Priorities
~~~~~~~~~~~~~~~~~~~~

+-------------------------------------------------+---------------------------+
| Title                                           | Owners                    |
+=================================================+===========================+
| Middleware upgrades Grafana 4.0 -> 7.0.1        | dougszu                   |
+-------------------------------------------------+---------------------------+
| `Kafka/InfluxDB Sharding`_                      | witek                     |
+-------------------------------------------------+---------------------------+
| Merge events-api into monasca-api               | adriancz                  |
+-------------------------------------------------+---------------------------+


High Priorities
~~~~~~~~~~~~~~~

+----------------------------------------------------+-------------------------+
| Title                                              | Owners                  |
+====================================================+=========================+
| `New thresholding engine`_                         | chaconpiza              |
+----------------------------------------------------+-------------------------+
| `Application Credentials`_                         |                         |
+----------------------------------------------------+-------------------------+
| Define Prometheus based architecture               |                         |
+----------------------------------------------------+-------------------------+
| Selenium Tests for Monasca-UI                      |                         |
+----------------------------------------------------+-------------------------+
| Middleware upgrades ELK 7.3.0 -> 7.7.0             | dougszu                 |
+----------------------------------------------------+-------------------------+
| Middleware upgrades Apache Kafka 2.0.1 -> 2.5.0    |                         |
+----------------------------------------------------+-------------------------+
| Middleware upgrades InfluxDB 1.7.6 -> 1.8.0        |                         |
+----------------------------------------------------+-------------------------+

Optional Priorities
~~~~~~~~~~~~~~~~~~~

+------------------------------------------+-------------------------+
| Title                                    | Owners                  |
+==========================================+=========================+
| Extend Monasca API                       |                         |
+------------------------------------------+-------------------------+
| Add Time and Times in Monasca UI         |                         |
+------------------------------------------+-------------------------+
| Monasca agents RPM Packaging             |                         |
+------------------------------------------+-------------------------+
| OpenStack Client Integration             |                         |
+------------------------------------------+-------------------------+
| Refresh Monasca transform engine         | dougsz                  |
+------------------------------------------+-------------------------+

Details
~~~~~~~

New thresholding engine
--------------------------------------------

`Faust library`_ has been evaluated and the prototype of the thresholding
engine based on this library has been implemented. The goal of this effort is
to implement the new thresholding engine for Monasca to replace Apache Storm
Java application.

.. _Faust library: https://faust.readthedocs.io

Kafka/InfluxDB Sharding
-----------------------

Story: https://storyboard.openstack.org/#!/story/2005620

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

Middleware upgrade
------------------

Story: https://storyboard.openstack.org/#!/story/2006768

