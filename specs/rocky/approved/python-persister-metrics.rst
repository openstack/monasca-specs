..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================================
Python Persister Performance Metrics Collection
===============================================

Story board: https://storyboard.openstack.org/#!/story/2001576

This defines the list of measurements for the metric upsert processing time and
throughput in Python Persister and provides a rest api to retrieve those
measurements.

Problem description
===================

The Java Persister, built on top of the DropWizard framework, provides a list
of internal performance related metrics, e.g., the total number of metric
messages that have been processed since the last service start up, the average,
min and max metric processing time etc. The Python Persister, on the other
hand, lacks such instrumentation. This presents a challenge to the operator
who wants to monitor, triage, and tune the Persister performance and to the
Persister performance testing tool that was introduced in Queens release. The
Cassandra Python Persister plugin depends on this feature for performance
tuning.

Use Cases
---------

- Use case 1: The developer instruments the defined performance metrics.

  There are two approaches towards the internal performance metrics. The first
  approach is in memory metering similar to the Java implementation. The data
  collection starts when the Persister service starts up and is not persisted
  through service restart. The second approach is to treat such measurement
  exactly the same as the "normal" metrics Monasca collects. The advantage is
  that such metrics will be persisted and rest apis are already available to
  retrieve the metrics.
  The list of Persister metrics includes:

  1. Total number of metrics upsert request received and completed on a given
     Persister service instance in the given period of time
  2. Total number of metrics upsert request received and completed on a
     process or thread in a given period of time (P2)
  3. The average, min, max metric request processing time in a given period of
     time for a given Persister service instance and process/thread.

- Use case 2: Retrieves persister performance metrics through rest api.

  The performance  metrics can be retrieved using the list metrics api in the
  Monasca API service.

Proposed change
===============

1. Monasca Persister

   - Python Persister integrates with monasca-statsd to send count and timer
     metrics
   - Persister conf to add properties for statsd

2. Persister performance benchmark tool adds support to retrieve the metrics
   from Monasca rest api source in addition to the DropWizard admin api.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

TBD, The statsd call to update counter and timer is expected to have small
performance impact.

Other deployer impact
---------------------

No change in deployment of the services.

Developer impact
----------------

None.

Implementation
==============

Assignee(s)
-----------

Contributors are welcome!

Primary assignee:

Other contributors:


Work Items
----------

1. Monasca Persister

   - Python Persister integrates with monasca-statsd to send count and timer
     metrics
   - Persister conf to add properties for statsd

2. Persister performance benchmark tool adds support to retrieve the metrics
   from Monasca rest api source in addition to the DropWizard admin api.


Dependencies
============

None

Testing
=======

- Set up a system, use JQuery to automate storing many metrics, check results.
  The tools to accomplish this testing can be found in monasca-persister/perf/


Documentation Impact
====================

The existing README.md in monasca-persister/perf describes the needed steps.
Some minor changes may need to be made to stay current.


References
==========

https://github.com/openstack/monasca-persister/tree/master/perf


History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Rocky
     - Introduced
   * - Stein
     - Revised with testing notes
