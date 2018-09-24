.. _stein-priorities:

=========================
Stein Project Priorities
=========================

List of priorities the Monasca drivers team is prioritizing in Stein.

The owners listed are responsible for tracking the status of that work and
helping get that work done. They are not the only contributors to this work,
and not necessarily doing most of the coding!

The implementation progress on these priorities and other identified important
tasks is tracked in `this board`_.

.. _this board: https://storyboard.openstack.org/#!/board/111

Essential Priorities
~~~~~~~~~~~~~~~~~~~~

+-----------------------------------------------+-----------------------------+
| Title                                         | Owners                      |
+===============================================+=============================+
| `Kafka client upgrade`_                       | witek                       |
+-----------------------------------------------+-----------------------------+
| `Monasca Events Agent`_                       | joadavis, aagate            |
+-----------------------------------------------+-----------------------------+
| `Merge Monasca APIs`_                         | dougsz                      |
+-----------------------------------------------+-----------------------------+
| `Add query endpoint for logs/events`_         | dougsz                      |
+-----------------------------------------------+-----------------------------+
| `Run under Python 3 by default`_              | adriancz, Dobroslaw         |
+-----------------------------------------------+-----------------------------+
| `Pre upgrade checks`_                         | joadavis                    |
+-----------------------------------------------+-----------------------------+

High Priorities
~~~~~~~~~~~~~~~

+---------------------------------------------+-------------------------+
| Title                                       | Owners                  |
+=============================================+=========================+
| Auto-scaling with Heat                      | witek                   |
+---------------------------------------------+-------------------------+
| `Metrics retention policy`_                 | joadavis                |
+---------------------------------------------+-------------------------+
| Documentation refresh                       |                         |
+---------------------------------------------+-------------------------+
| Deployment in OpenStack Helm                | srwilkers               |
+---------------------------------------------+-------------------------+
| Integration with Watcher                    | yushiro                 |
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

Merge Monasca APIs
------------------

The goal is to merge all Monasca APIs into a single unified API to reduce
maintenance overhead, make it easier for developers to add new features and
improve the user experience.

Monasca Events Agent
--------------------

The goal is to extend Monasca Ceilometer project and add a new events publisher
which will publish Openstack notifications (or events) to Monasca Events API.

Add query endpoint for logs/events
----------------------------------

`Add support`_ for querying ElasticSearch via the Monasca API to support tenant
scoped access to logs and events. This should include accessing the logs via
Grafana.

.. _Add support: https://blueprints.launchpad.net/monasca/+spec/log-query-api

Run under Python 3 by default
-----------------------------

As OpenStack Technical Committee agreed in the `Python2 Deprecation Timeline`_
resolution, the next phase of our adoption of Python 3 is to begin running all
jobs using Python 3 by default and only using Python 2 to test operating under
Python 2 (via unit, functional, or integration tests). This goal describes the
activities needed to move us to this `python 3 first`_ state.

.. _Python2 Deprecation Timeline: https://governance.openstack.org/tc/resolutions/20180529-python2-deprecation-timeline.html#python2-deprecation-timeline
.. _Python 3 first: https://governance.openstack.org/tc/goals/stein/python3-first.html

Pre upgrade checks
------------------

The goal is to provide an `upgrade check command`_ which would perform any
upgrade validation that can be automated.

.. _upgrade check command: https://governance.openstack.org/tc/goals/stein/upgrade-checkers.html

Metrics retention policy
------------------------

The goal is to add a new API for managing the mapping of metrics to TTL values.
