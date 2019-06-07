.. _train-priorities:

=========================
Train Project Priorities
=========================

List of priorities the Monasca drivers team is prioritizing in Train.

The owners listed are responsible for tracking the status of that work and
helping get that work done. They are not the only contributors to this work,
and not necessarily doing most of the coding!

The implementation progress on these priorities and other identified important
tasks is tracked in `this board`_.

.. _this board: https://storyboard.openstack.org/#!/board/141

Essential Priorities
~~~~~~~~~~~~~~~~~~~~

+-------------------------------------------------+---------------------------+
| Title                                           | Owners                    |
+=================================================+===========================+
| `Kafka client upgrade`_                         | witek                     |
+-------------------------------------------------+---------------------------+
| `Merge Monasca APIs`_                           | adriancz                  |
+-------------------------------------------------+---------------------------+
| `Middleware upgrade`_                           | dougsz                    |
+-------------------------------------------------+---------------------------+
| `Thresholding engine replacement (tech prev.)`_ |                           |
+-------------------------------------------------+---------------------------+
| `PDF generation for documentation`_             |                           |
+-------------------------------------------------+---------------------------+

High Priorities
~~~~~~~~~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| `Application credentials (Grafana)`_        | dougsz                  |
+---------------------------------------------+-------------------------+
| `Application credentials (agent)`_          |                         |
+---------------------------------------------+-------------------------+
| `Documentation refresh`_                    | joadavis                |
+---------------------------------------------+-------------------------+
| `Java Persister deprecation`_               | joadavis                |
+---------------------------------------------+-------------------------+

Optional Priorities
~~~~~~~~~~~~~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| `Monasca Events Agent`_                     |                         |
+---------------------------------------------+-------------------------+
| New query language                          |                         |
+---------------------------------------------+-------------------------+
| OpenStack CLI                               | sc                      |
+---------------------------------------------+-------------------------+
| Reuse Prometheus dashboards                 |                         |
+---------------------------------------------+-------------------------+
| Vitrage integration                         | chaconpiza              |
+---------------------------------------------+-------------------------+

Backlog
~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| Sharding model for InfluxDB                 | dougsz                  |
+---------------------------------------------+-------------------------+
| OpenStack Helm                              |                         |
+---------------------------------------------+-------------------------+
| OpenStack Ansible                           | sc                      |
+---------------------------------------------+-------------------------+
| Senlin integration                          |                         |
+---------------------------------------------+-------------------------+
| Gnocchi support                             |                         |
+---------------------------------------------+-------------------------+

Details
~~~~~~~

Kafka client upgrade
--------------------

Currently, in all Python Monasca components, the copy of `kafka-python` library
in version 0.9.5 (released on Feb 16, 2016) is used. Sticking with the old
frozen client version is also unacceptable in terms of security. The goal is to
upgrade the Apache Kafka client to `confluent-kafka-python`. This will
dramatically improve the performance and reliability.

Story: https://storyboard.openstack.org/#!/story/2003705

Merge Monasca APIs
------------------

The goal is to merge all Monasca APIs into a single unified API to reduce
maintenance overhead, make it easier for developers to add new features and
improve the user experience.

Story: https://storyboard.openstack.org/#!/story/2003881

Middleware upgrade
------------------

We want to change the general approach and try to use the newest (stable)
versions of software available. The beginning of the cycle is the good time
point to upgrade components such as e.g.: Apache Kafka, InfluxDB, Apache
Storm, ELK.

Story: https://storyboard.openstack.org/#!/story/2005624

Thresholding engine replacement (tech prev.)
--------------------------------------------

The goal of this task is to provide the technical preview of the new
component replacing the current thresholding engine.

Story: https://storyboard.openstack.org/#!/story/2005598

PDF generation for documentation
--------------------------------

This is the community wide goal.

https://governance.openstack.org/tc/goals/train/pdf-doc-generation.html

Application credentials (Grafana)
---------------------------------

`Keystone appliction credentials <https://docs.openstack
.org/keystone/latest/user/application_credentials.html>`_ offer the mechanism
to allow applications to authenticate to Keystone. The ability to specify
`access rules <http://specs.openstack
.org/openstack/keystone-specs/specs/keystone/stein/capabilities-app-creds
.html>`_ for application credentials is being developed and will be
released in the Train cycle.

The goal of this story is to add application credentials support in
monasca-grafana-datasource. The access rules should be limited to only
reading the measurements from Monasca. It will allow storing these
credentials directly in the datasource without the security risk of revealing
the OpenStack user's password. It will also decouple the datasource from
Grafana's authentication.

Story: https://storyboard.openstack.org/#!/story/2005623

Application credentials (agent)
-------------------------------

`Keystone appliction credentials <https://docs.openstack
.org/keystone/latest/user/application_credentials.html>`_ offer the mechanism
to allow applications to authenticate to Keystone. The ability to specify
`access rules <http://specs.openstack
.org/openstack/keystone-specs/specs/keystone/stein/capabilities-app-creds
.html>`_ for application credentials is being developed and will be
released in the Train cycle.

The goal of this story is to add application credentials support in
*monasca-agent*. This will prevent the security risk of revealing OpenStack
user's password when installing the agent on the tenants environment. The
access rules of these application credentials should be limited to posting
measurements. *monasca-setup* should be extended to automatically generate such
credentials and save them in configuration file if needed.

Documentation refresh
---------------------

Story: https://storyboard.openstack.org/#!/story/2005625

Java Persister deprecation
--------------------------

Story: https://storyboard.openstack.org/#!/story/2005628

Monasca Events Agent
--------------------

The goal is to extend Monasca Ceilometer project and add a new events publisher
which will publish Openstack notifications (or events) to Monasca Events API.

Story: https://storyboard.openstack.org/#!/story/2003023
