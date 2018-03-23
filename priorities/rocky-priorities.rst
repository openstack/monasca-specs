.. _rocky-priorities:

=========================
Rocky Project Priorities
=========================

List of priorities the Monasca drivers team is prioritizing in Rocky.

The owners listed are responsible for tracking the status of that work and
helping get that work done. They are not the only contributors to this work,
and not necessarily doing most of the coding!

Essential Priorities
~~~~~~~~~~~~~~~~~~~~

+-----------------------------------------------+-----------------------------+
| Title                                         | Owners                      |
+===============================================+=============================+
| Kafka upgrade                                 |                             |
+-----------------------------------------------+-----------------------------+
| Alembic migrations                            | jgr, amofakhar              |
+-----------------------------------------------+-----------------------------+
| Metrics retention policy                      | jgu                         |
+-----------------------------------------------+-----------------------------+
| Monasca Transformer refresh                   | aagate, joadavis            |
+-----------------------------------------------+-----------------------------+
| `Run API under WSGi`_                         | witek                       |
+-----------------------------------------------+-----------------------------+
| `Support Python 3.5`_                         | witek, sc                   |
+-----------------------------------------------+-----------------------------+
| Enable mutable configuration                  |                             |
+-----------------------------------------------+-----------------------------+
| `Policy in Code`_                             | amofakhar                   |
+-----------------------------------------------+-----------------------------+

High Priorities
~~~~~~~~~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| `Templating webhook notifications`_         | dougsz                  |
+---------------------------------------------+-------------------------+
| :ref:`service-agent-domain`                 | jgr                     |
+---------------------------------------------+-------------------------+
| `Add Monasca publisher to Ceilometer`_      | joadavis, aagate        |
+---------------------------------------------+-------------------------+
| Alarm grouping, silencing, inhibition       | witek                   |
+---------------------------------------------+-------------------------+
| Documentation refresh                       |                         |
+---------------------------------------------+-------------------------+

Optional Priorities
~~~~~~~~~~~~~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| New agent plugins for OpenStack             |                         |
+---------------------------------------------+-------------------------+
| Cross-project integrations                  |                         |
+---------------------------------------------+-------------------------+
| Monasca Query Language                      |                         |
+---------------------------------------------+-------------------------+
| Create Docker images from OpenStack repos   |                         |
+---------------------------------------------+-------------------------+
| `Kolla deployment`_                         | dougsz                  |
+---------------------------------------------+-------------------------+
| `Query logs pipeline`_                      | dougsz                  |
+---------------------------------------------+-------------------------+
| New monasca-thresh                          |                         |
+---------------------------------------------+-------------------------+
| Monasca-persister performance improvements  | sgrasley, jgu           |
+---------------------------------------------+-------------------------+

Details
~~~~~~~

Run API under WSGi
-----------------------------------

This is a community-wide release goal for Pike. `The goal`_ is to:

* Provide WSGI application script files.
* Switch devstack jobs to deploy Monasca APIs under uwsgi with Apache acting as
  a front end proxy.

.. _The goal: https://governance.openstack.org/tc/goals/pike/deploy-api-in-wsgi.html

Support Python 3.5
-----------------------------------

This is a community-wide release goal for Pike. The goal is to
support, test and running with `Python 3.5`_.

.. _Python 3.5: https://governance.openstack.org/tc/goals/pike/python35.html

Policy in Code
-------------------------------

The goal is to register and document default `policies`_ for the APIs in code.

.. _policies: https://governance.openstack.org/tc/goals/queens/policy-in-code.html

Add Monasca publisher to Ceilometer
-----------------------------------

Monasca-Ceilometer (aka. Ceilosca) code currently exists in its own project.
This is for historical reasons.  With changes in Ceilometer and the
Telemetry project, it may be possible to have the Monasca publisher from
monsasca-ceilometer `merged into the Ceilometer`_ repository.  This could reduce
future workload in maintenance.

.. _merged into the Ceilometer: https://storyboard.openstack.org/#!/story/2001239

Templating webhook notifications
--------------------------------

Improve the quality of notifications generated from alerts. We want notifications
to be informative, concise and flexible.

Kolla deployment
----------------

Add support for deploying Monasca in Docker containers using the OpenStack Kolla
project. This change will support deploying Monasca in a high availability
configuration. Blueprints exist for `containers`_ and the `Ansible roles`_ to deploy
them.

.. _containers: https://blueprints.launchpad.net/kolla/+spec/monasca-containers
.. _Ansible roles: https://blueprints.launchpad.net/kolla-ansible/+spec/monasca-roles

Query logs pipeline
-------------------

`Add support`_ for querying ElasticSearch via the Monasca Log API to support tenant
scoped access to logs. This should include accessing the logs via Grafana.

.. _Add support: https://blueprints.launchpad.net/monasca/+spec/log-query-api
