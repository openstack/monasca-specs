..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================================
Metric Retention Policy
================================================

Story board: https://storyboard.openstack.org/#!/story/2001576

Metric retention policy must be in place to avoid disk being filled up.
Retention period should be adjustable for different types of metrics, e.g.,
monitoring vs. metering or aggregate vs. raw meters.

Problem description
===================

In a cloud of 200 compute hosts, there can be up to one billion metrics
generated daily. The time series database disks will be filled up in months
if not weeks if old metric data is not purged regularly. The retention
requirement can be different based on the type of the metrics and the usage
model. For example, the customer may want to preserve the metering metrics
for months or years, while s/he has no interest in more than a week old
monitoring metrics. Some customers' billing system may pull the metering data
on a daily base which could eliminate the need of longer retention of metering
metrics. Monasca needs to support metric retention policy that can be tailored
per metric or metric type.

Use Cases
---------

- Use case 1:
  Installer sets a default TTL value in configuration.  At installation time,
  a default TTL (time to live) value is specified in the configuration for
  monasca-api and is used as the default retention policy.

  The default retention policy is applied if a metric doesn't match another
  retention policy. This default retention is generally a shorter period of
  time and may be used for the common monitoring metrics.

- Use case 2:
  Installer loads a set of metric to TTL mappings (retention policies), which
  is stored in the Monasca API data store (mysql database).  These mappings may
  be provided in a JSON structure.  This is intended to be useful for bootstrap
  or restore from backup.

- Use case 3:
  Monasca API receives new metric (regardless of source).  Metric is mapped to
  a dictionary to determine TTL (or default value used if no match).  TTL is
  passed with metric value on to the Persister for storage in TSDB.

  Note that the use cases for monasca-agent to post metrics are unchanged, just
  the processing at Monasca API then the API to Persister message.]

  The Monasca Persister then stores the metric and specifies the TTL to the
  TSDB configured (i.e. InfluxDB or Cassandra).

- Use case 4:
  Operator uses Monasca CLI to specify (or modify) a TTL value for a metric
  match string. Match string could be specific, such as "cpu.user_perc" or a
  wildcard string, such as "image.*".  CLI posts request to Monasca TTL API,
  where it is validated then stored in database.

- Use case 5:
  Operator uses Monasca CLI to GET the dictionary of metric:TTL mappings.
  This can be used to export the list for backup or verification.

- Use case 6 (optional):
  Operator uses Monasca UI to accomplish use case 4 or 5


Proposed change
===============

1. Monasca API:
   Add a new API for managing the mapping of metrics to TTL values.
   See the `REST API impact`_ section below.

   Add storage for the mapping in the MySQL database. This is to allow
   all instances of Monasca API to share the configuration dynamically.
   *TBD* - Create a schema for storing the metric:TTL dictionary.

   A policy precedence needs to be defined.  It is possible that more than
   one retention policy may apply to a given meter, so a clear precedence
   needs to be defined to determine which TTL value to apply.
   *TBD* - a few concrete examples.

2. Monasca Persister:
   Persister reads the default retention policy setting from the service
   configuration file in the influxDbConfiguration and cassandraDbConfiguration
   section.
   ::

     # Retention policy may be left blank to indicate default policy.
     # Unit is days
     retentionPolicy: 7

   It may be convenient to allow specifying a unit with the policy value.  For
   example "7d" for 7 days or "3m" for 3 months.

   It will retrieve the TTL property in the incoming metric message. If not set,
   the TTL value from the default retention policy will used instead.

   It is expected with the addition of this Metrics Retention feature that the
   default retentionPolicy value would be set to a low value, and that metrics
   that are to be kept longer would be called out specifically through the
   Retention API and appropriate values set.

   The TTL is set in the parameterized database query when persisting the metrics
   into the time series database, including both Cassandra and InfluxDB.
   *TBD* - exact call structures for each TSDB.

   Note that this does mean that each storage back end would need to have code
   customized in the persister to support passing the TTL value.  This may also
   be possible for ElasticSearch, though that is not part of this initial spec.

3. Monasca CLI (optional):
   A new CLI feature could be created to simplify getting the list of TTL
   mappings or posting an update to a TTL mapping.  This would need Keystone
   authentication, and would use the existing 'monasca' CLI authentication.

4. Monasca UI (optional):
   A new feature could be added to the Monasca UI that would allow a Cloud
   Operator to view and edit the list of TTL mappings.
   Bonus points for allowing the UI to have sample metrics and simulate the
   mapping on the page.

Alternatives
------------

The original proposal was to have monasca-transform, monasca-ceilometer, and
monasca-agent each keep a TTL default setting and have a property to allow
specifying a TTL per metric.  This would have also required a change to the
Monasca API to add an optional TTL to the metric POST listener.

While this would have been simpler to implement in the Monasca API, the
additional work to change all the services that originate metrics made this
alternative not as appealing.


Another alternative would be to implement a new Monasca Retention API as
outlined, but not include dimensions for the metrics. This would allow a much
simpler data structure of key:value pairs, with the key being the unique match
string and the value the standardized TTL value.  While the implementation
would be much simpler, it is felt that the additional power of having match
dimensions would be beneficial.


Data model impact
-----------------

The Monasca API data model will need to be extended to store the metric to
TTL mappings (retention policies).
*TBD* - schema

REST API impact
---------------

A new metric retention API endpoint would be added to Monasca API.

URL: /v2.0/metrics-retention

Method: GET
  A GET request will return the current list of metric retention policies.
  Examples::

    Empty list (default retention used for all metrics)
    []

    Simple list
    [
      {
        match: "cpu.user_perc",
        dimensions: {"host": "node1"},
        retentionPolicy: "7d"
      },
      {
        match: "cpu.stolen_perc",
        dimensions: {},
        retentionPolicy: "7d"
      }
    ]

Method: PUT
  The PUT method is used for all create/update/delete methods on the metric
  retention policy list.  Any list of metrics PUT to the API will be merged
  with the existing list.  Single entries will also be supported.

  JSON structure for PUT/GET to Retention API::

    {
      match: "cpu.user_perc",
      dimensions: {},
      retentionPolicy: "7d"
    }

  TBD: do we support adding a character for time unit?  Will it be confusing to
  PUT "1d" and GET back "86400"?

  Special case: to delete a retention policy, give a retentionPolicy value of
  None and it will be removed from the list.
  ::

    {
      match: "cpu.user_time",
      dimensions: {},
      retentionPolicy: None
    }

  Additionally, a list of retention policy items may be PUT, with the format
  matching the response from GET. Each item in the list will be compared to
  existing metric policies (match string and dimensions). If an exact match is
  found, the retentionPolicy value will be replaced. Otherwise, the new item is
  added to the list.
  (This is intended to make bootstrap or restore from backup easier)


The communication from Monasca API to Persister would have the TTL value
added as a parameter.

NOTE: Care should be taken in defining the REST API path, as Gnocchi uses
"/metric", which may be confusing to some users.


Security impact
---------------

None.  Security measures already in place for the Monasca API would remain.

Other end user impact
---------------------

None for most users, as access to the Monasca Metrics API is restricted to
Cloud Operators.
A Cloud Operator would have a new responsibility to configure retention for
the metrics.

A future discussion could be had about whether a tenant user should be granted
the ability to set their own retention policies, but generally the Cloud
Operator is responsible for ensuring there are sufficient resources to meet the
retention requirements.

Performance Impact
------------------

This feature has no direct impact on the write throughput. However, it allows
the user to enable shorter retention period for monitoring metrics which
can potentially improve the read performance for the queries that involves
search, grouping and filtering when there are less metrics in the table.  This
improves the storage footprint.

Depending on how complex the metric retention match string gets there could be
some performance impact. *TBD*

Other deployer impact
---------------------

No change in deployment of the services.
The service could be deployed with simply a default TTL value in configuration.
If the operator desires, at install time a complete list of TTL values could
be loaded as part of the installation process once the Monasca API is running.

For planning, the user now has the option to specify a shorter retention period
for monitoring metrics or even per metric or metric category. The disk size
should be calculated based upon the retention policy accordingly.

Developer impact
----------------

Monasca agent plugin developers should be aware of the new TTL property
now available to them. It is an optional property that is only needed if a
different TTL value than the default retention policy in the Persister service
is needed.


Implementation
==============

Assignee(s)
-----------

Contributors are welcome!

Primary assignee:


Other contributors:


Work Items
----------

* Add new metrics-retention API endpoint to Monasca API

* Add code to match all incoming metrics to the Monasca API with the appropriate
  retention policy (or default)

* Add TTL in seconds as a parameter to the request from Monasca API to
  Persister

* Create a CLI

  * PUT of updated retention policy(ies)
  * GET of the list

* Determine correct precedence for retention policies that overlap, and clearly
  document with examples.


Dependencies
============

Dependent on retention policy support in the TSDB storage.  Both Cassandra
and InfluxDB support specifying a retention policy.


Testing
=======

Unit testing
  Unit tests in the Monasca API should be written for the scenarios of defining
  a TTL for each metric.

  * Metric received, no matching retention policy found, default policy used
  * Metric received, one exact matching metric retention policy found, matching
    policy parameter passed to Persister call
  * Metric received, more than one matching policy, correct precedent determined
    and appropriate policy parameter passed to Persister call

  Monasca Persister will also need unit tests to verify the passed-in value is
  passed on to the TSDB retention method call, and to handle the case of a missing
  TTL parameter.  We may decide that the TTL parameter is optional then a global
  default TTL value should be used.

Functional testing
  Functional testing is more involved, as one way to test would be to trigger some
  metrics, have them stored in the TSDB, then wait for the TTL value to expire and
  verify the metric is removed correctly. More thought and definition is needed
  to define what is appropriate and possible (i.e. to not retest features of the
  TSDB).

Documentation Impact
====================

Operators who use Monasca would need documentation to describe the format of
the new API and recommended usage.  This may include guidelines on how to set
a low default and to choose which metrics should be kept longer.  The default
TTL value as set in a config file should also be documented.

References
==========

* Links

  * Stein PTG discussion - https://etherpad.openstack.org/p/monasca-ptg-stein

* Glossary

  * TTL - short for Time to Live, a setting in TSDB that defines when an item
    (in this case a metric) will be cleaned out.

  * TSDB - Time Series Database, such as InfluxDB or Cassandra.


History
=======

Optional section intended to be used each time the spec is updated to describe
new design, API or any database schema updated. Useful to let reader understand
what's happened along the time.

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Queens
     - Introduced
