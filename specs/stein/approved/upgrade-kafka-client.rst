..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================
Upgrade Apache Kafka client
===========================

Include the URL of your story:

https://storyboard.openstack.org/#!/story/2003705

Currently in all Python Monasca components the copy of `kafka-python` library
in version 0.9.5 (released on Feb 16, 2016) is used [1]_. This specification
describes the process of upgrading the Apache Kafka client to
`confluent-kafka-python` [2]_. This will improve the performance and
reliability. Sticking with the old frozen client version is also unacceptable
in terms of security.

Problem description
===================

The use of `KeyedProducer` and `SimpleConsumer` in `kafka-python` library has
been deprecated as of version 1.0.0 [3]_. Further use of this code poses a
security risk. Additionally, profiling of ``monasca-persister`` has shown that
most of the time is spent during the consumption of Kafka messages [7]_. Thus,
there is a big potential on improving overall Monasca performance by upgrading
the used Kafka client.

Proposed change
===============

The wiki page hosted by Apache Software Foundation lists available Python
clients [4]_. There are currently three actively maintained and supported
clients: `confluent-kafka-python`, `kafka-python` and `pykafka`. Several
benchmarks have shown [5]_, [6]_ that the client maintained by Confluent is
both the fastest and most complete.

There is significant performance improvement when using asynchronous producer
(~50x). Sending messages asynchronously will require more care to avoid
duplicating the persisted data but performance gain justifies that.

`confluent-kafka-python` is also the only client which offers support for
Apache Avro serialization which reduces the size of messages and thus
additionally speeds up communication.

The proposed change includes using:

* `confluent-kafka-python` library
* in asynchronous mode

Code changes will affect following components:

* monasca-common
* monasca-{log,event}-api
* monasca-persister
* monasca-notification
* monasca-transform

Java components (`monasca-thresh` and `monasca-persister`) are out of scope of
this specification. Client upgrading in these components should be handled
separately.

This client has an external dependency on `librdkafka`, a finely tuned C
client.

Alternatives
------------

* `pykafka`
* new version of `kafka-python`
* use synchronous mode

Data model impact
-----------------

No data model impact.

REST API impact
---------------

No REST API impact.

Security impact
---------------

This change will improve the security because of removing the deprecated and
unmaintained code.

Other end user impact
---------------------

No end user impact.

Performance Impact
------------------

This change should dramatically improve the performance of the complete
solution. In particular performance of `monasca-persister` and `monasca-api` is
expected to improve.

Other deployer impact
---------------------

New libraries should be packaged and deployed:

* `confluent-kafka-python`
* `librdkafka`

Developer impact
----------------

`confluent-kafka-python` has to be used instead of `kafka-python` in all
affected components.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  witek

Other contributors:
  <>

Work Items
----------

* remove code using `pykafka`
* remove `pykafka` from requirements and lower-constraints
* add `confluent-kafka-python` to global-requirements
* implement common routines in `monasca-common`
* use new code in:
  * monasca-{log,events}-api
  * monasca-persister
  * monasca-notification
  * monasca-transform
* delete old deprecated code

Dependencies
============

New packages have to be build for:

* `confluent-kafka-python`
* `librdkafka`

Testing
=======

We should test the implementation using existing integration tests (tempest).
Additionally we should test the scenario when the producer fails to receive
response from Kafka for some of the messages in the bulk. It should be avoided
that duplicate entries are created in the database.

The implantation should be followed by executing following tests on the
complete stack:

* stress
* endurance
* performance

Documentation Impact
====================

No documentation impact.

References
==========

.. [1] https://github.com/dpkp/kafka-python/releases/tag/v0.9.5
.. [2] https://github.com/confluentinc/confluent-kafka-python
.. [3] https://github.com/dpkp/kafka-python/blob/master/docs/changelog.rst#100-feb-15-2016
.. [4] https://cwiki.apache.org/confluence/display/KAFKA/Clients#Clients-Python
.. [5] https://github.com/monasca/monasca-perf/blob/master/kafka_python_client_perf/monascaInvestigationKafkaPythonAPIs.md
.. [6] http://activisiongamescience.github.io/2016/06/15/Kafka-Client-Benchmarking/
.. [7] http://git.openstack.org/cgit/openstack/monasca-persister/commit/?id=a7112fd30bd545dd850e0e267dcceb9ea27551ad


History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Stein
     - Introduced
