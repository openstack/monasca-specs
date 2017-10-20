.. _queens-priorities:

=========================
Queens Project Priorities
=========================

List of priorities the Monasca drivers team is prioritizing in Queens.

The owners listed are responsible for tracking the status of that work and
helping get that work done. They are not the only contributors to this work,
and not necessary doing most of the coding!

Essential Priorities
~~~~~~~~~~~~~~~~~~~~

+---------------------------------------------+-----------------------------+
| Title                                       | Owners                      |
+=============================================+=============================+
| Cassandra support                           | jgu                         |
+---------------------------------------------+-----------------------------+
| `Add Monasca publisher to Ceilometer`_      | joadavis, aagate            |
+---------------------------------------------+-----------------------------+
| `Fix Keystone authentication for Grafana`_  | rhochmuth, witek, Dobroslaw |
+---------------------------------------------+-----------------------------+
| Alarm grouping, silencing, inhibition       | Andrea Adams, rhochmuth     |
+---------------------------------------------+-----------------------------+
| `Run API under WSGi (Community Goal)`_      | kornicameister, witek       |
+---------------------------------------------+-----------------------------+
| `Support Python 3.5 (Community Goal)`_      | witek, sc                   |
+---------------------------------------------+-----------------------------+
| `Split Tempest Plugins (Community Goal)`_   |                             |
+---------------------------------------------+-----------------------------+

High Priorities
~~~~~~~~~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| Service Domain for Self Service Agent Users | jgr                     |
+---------------------------------------------+-------------------------+
| Replace python-kafka with pykafka           |                         |
+---------------------------------------------+-------------------------+
| Metrics retention policy                    |                         |
+---------------------------------------------+-------------------------+
| `Persisting Events`_                        | witek                   |
+---------------------------------------------+-------------------------+
| Monasca Query Language                      |                         |
+---------------------------------------------+-------------------------+
| `Policy in Code (Community Goal)`_          | jgr                     |
+---------------------------------------------+-------------------------+

Optional Priorities
~~~~~~~~~~~~~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| `3-nodes cluster with Docker Compose`_      | witek                   |
+---------------------------------------------+-------------------------+
| Add message attributes to Log API           | koji                    |
+---------------------------------------------+-------------------------+

Details
~~~~~~~

Add Monasca publisher to Ceilometer
-----------------------------------

Monasca-Ceilometer (aka. Ceilosca) code currently exists in its own project.
This is for historical reasons.  With changes in Ceilometer and the
Telemetry project, it may be possible to have the Monasca publisher from
monsasca-ceilometer merged in to the Ceilometer repository.  This could reduce
future workload in maintenance.

.. _ceilosca merge storyboard: https://storyboard.openstack.org/#!/story/2001239

.. _grafana-auth:

Fix Keystone authentication for Grafana
---------------------------------------

The current implementation of Keystone authentication for Grafana is maintained
in the `forked repository`_. Due to upstream changes in Grafana major
refactoring is required to rebase the fork with newest Grafana code.

The goal is to contribute Keystone authentication (or generic pluggable
authentication mechanism) to Grafana upstream. If not possible, the current
fork should be refactored to allow its further maintenance.

.. _forked repository: https://github.com/monasca/grafana

Run API under WSGi (Community Goal)
-----------------------------------

This is a community-wide release goal for Pike. The goal is to
support, and test, running `WSGI`_.

.. _WSGI: https://governance.openstack.org/tc/goals/pike/deploy-api-in-wsgi.html

Support Python 3.5 (Community Goal)
-----------------------------------

This is a community-wide release goal for Pike. The goal is to
support, and test, running with `python 3.5`_.

.. _python 3.5: https://governance.openstack.org/tc/goals/pike/python35.html

Split Tempest Plugins (Community Goal)
--------------------------------------

This goal is to make sure we always use a `separate python project`_ for
monasca-api, monasca-log-api and monasca-events-api tempest plugins.

.. _separate python project: https://governance.openstack.org/tc/goals/queens/split-tempest-plugins.html

Policy in Code (Community Goal)
-------------------------------

The goal is to register and document default `policies`_ for the APIs in code.

.. _policies: https://governance.openstack.org/tc/goals/queens/policy-in-code.html

Persisting Events
-----------------

The goal is to provide the `pipeline`_ for persisting OpenStack notifications
and/or events from external systems to the database, e.g. Elasticsearch.

.. _pipeline: https://storyboard.openstack.org/#!/story/2001112

3-nodes cluster with Docker Compose
-----------------------------------

The goal is to provide an easy and simple way of deploying Monasca in a `static
3-nodes cluster`_ with Docker containers without using cluster management layer
like Kubernetes or Docker Swarm.

.. _static 3-nodes cluster: https://github.com/monasca/monasca-docker/issues/154
