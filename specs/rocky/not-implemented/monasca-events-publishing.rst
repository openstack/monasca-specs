..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================================
Monasca Events Publishing - from Ceilometer
================================================

https://storyboard.openstack.org/#!/story/2003023

Monasca Events API [1] was developed to store Openstack Notification data in Elasticsearch. There is
still a need to collect and publish Openstack Notifications to Monasca Events API. Monasca
Ceilometer project[2] currently publishes ceilometer samples[3] to Monasca API. We are proposing to
extend Monasca Ceilometer project and add a new events publisher which will publish Openstack
notifications (or events)[3] to Monasca Events API.

UPDATE: This spec is being superceded by the ../../stein/approved/monasca-events-listener.rst spec,
but is kept here for reference.


Problem description
===================

All Openstack services generate a lot of notifications or events which contain large amounts of
operational and state information about the service and its resources. This notification data is not
currently available in Monasca.

Ceilometer data processing pipeline[3] provides an extensible mechanism of publishing samples and
events using a custom publisher.  Ceilometer samples represent a quantity that can be measured (for
e.g. the size of a volume) and events represent an occurrence of an event and do not have any
associated quantity (e.g. volume was created).

Monasca Ceilometer project currently provides a samples publisher. Monasca Ceilometer samples
publisher converts Ceilometer samples to Monasca Metrics format which are then published to Monasca
API. There is no corresponding events publisher to Monasca yet.

By adding an event publisher to Monasca Ceilometer project we could take advantage of Ceilometer's
event publishing mechanism to publish events to Monasca Events API.

Ceilometer consists of different data collection components - namely Polling Agent, Notification
Agent and Compute Agent. (Please see [7] for System Architecture diagram) Ceilometer also has a data
storage and retrieval component, which would be Monasca in our case.

Samples publisher and new proposed events publisher run within Ceilometer's Notification Agent
component and are part of Notifcation Agent's data processing pipeline. Monasca
Ceilometer presumes the need to install and deploy Ceilometer Notification Agent component (doesn't
need Polling Agent or Compute Agent deployed) on all the control nodes. Ceilometer Notification
Agent is Highly Available (HA) and can run on multiple nodes. We will have to evaluate its
performance in terms of scaling for events, but we haven't run into performance/scale problems
with current samples publisher.


Use Cases
---------

#. Openstack notification data would be stored in Elasticsearch
   via the Monasca Events API

   Example sequence from Nova notification to Monasca API
   #. Nova completes the creation of a VM
   #. Nova generates a Notification message to oslo.messaging
   #. Ceilometer Notification agent receives the Notification message
   #. Ceilometer translates the Notification to a Monasca API format according to the configuration
   #. Ceilometer Event Publisher publishes formatted Notification to Monasca Events API
   #. Monasca Events API receives and validates formatted Notification
   #. Monasca Events stores event Notification in configured Elasticsearch instance


Proposed change
===============

#. Openstack Notifications consist of envelope and payload fields

   Example Openstack Notification data format:

   .. code-block:: javascript

        {
            "_context_auth_token": "42630b3ea13242fcad20e0a92d0207f1",
            "_context_domain": null,
            "_context_instance_lock_checked": false,
            "_context_is_admin": true,
            "_context_project_domain": null,
            "_context_project_id": "a4f77",
            "_context_project_name": "admin",
            "_context_quota_class": null,
            "_context_read_deleted": "no",
            "_context_read_only": false,
            "_context_remote_address": "192.168.245.4",
            "_context_request_id": "req-5948338c-f223-4fd8-9249-8769f7a3e460",
            "_context_resource_uuid": null,
            "_context_roles": [
                "monasca-user",
                "admin",
                "KeystoneAdmin"
            ],
            "_context_service_catalog": [
                {
                    "endpoints": [
                        {
                            "adminURL": "http://192.168.245.8:8776/v2/a4f77",
                            "internalURL": "http://192.168.245.8:8776/v2/a4f77",
                            "publicURL": "http://192.168.245.9:8776/v2/a4f77",
                            "region": "region1"
                        }
                    ],
                    "name": "cinderv2",
                    "type": "volumev2"
                },
                {
                    "endpoints": [
                        {
                            "adminURL": "http://192.168.245.8:8776/v1/a4f77",
                            "internalURL": "http://192.168.245.8:8776/v1/a4f77",
                            "publicURL": "http://192.168.245.9:8776/v1/a4f77",
                            "region": "region1"
                        }
                    ],
                    "name": "cinder",
                    "type": "volume"
                }
            ],
            "_context_show_deleted": false,
            "_context_tenant": "a4f77",
            "_context_timestamp": "2015-09-18T20:54:23.468522",
            "_context_user": "be396488c7034811a200a3cb1d103a28",
            "_context_user_domain": null,
            "_context_user_id": "be396488c7034811a200a3cb1d103a28",
            "_context_user_identity": "be396488c7034811a200a3cb1d103a28 a4f77 - - -",
            "_context_user_name": "admin",
            "_unique_id": "ff9699d587bf4283a3c367ab88be1541",
            "event_type": "compute.instance.create.start",
            "message_id": "c6149ba1-34b3-4367-b8c2-b1d6f073742d",
            "payload": {
                "access_ip_v4": null,
                "access_ip_v6": null,
                "architecture": null,
                "availability_zone": null,
                "cell_name": "",
                "created_at": "2015-09-18 20:55:25+00:00",
                "deleted_at": "",
                "disk_gb": 1,
                "display_name": "testeee",
                "ephemeral_gb": 0,
                "host": null,
                "hostname": "testeee",
                "image_meta": {
                    "base_image_ref": "df0c8",
                    "container_format": "bare",
                    "disk_format": "qcow2",
                    "min_disk": "1",
                    "min_ram": "0"
                },
                "image_name": "glanceaaa3",
                "image_ref_url": "http://192.168.245.5:9292/images/df0c8",
                "instance_flavor_id": "1",
                "instance_id": "abd2ef5c-0381-434a-8efc-d7b39b28a2b6",
                "instance_type": "m1.tiny",
                "instance_type_id": 4,
                "kernel_id": "",
                "launched_at": "",
                "memory_mb": 512,
                "metadata": {},
                "node": null,
                "os_type": null,
                "progress": "",
                "ramdisk_id": "",
                "reservation_id": "r-1ghilddw",
                "root_gb": 1,
                "state": "building",
                "state_description": "",
                "tenant_id": "a4f77",
                "terminated_at": "",
                "user_id": "be396488c7034811a200a3cb1d103a28",
                "vcpus": 1
            },
            "priority": "INFO",
            "publisher_id": "compute.ccp-compute0001-mgmt",
            "timestamp": "2015-09-18 20:55:37.639023"
        }

#. All the fields with the prefix of '_context" are the envelope fields, the
   other interesting fields are

   #. 'message_id' - notification identifier
   #. 'payload' - contains most of the relevant and useful information in JSON format
   #. 'priority' - notification priority
   #. 'publisher_id' - notification publisher
   #. 'timestamp' - notification timestamp

#. Ceilometer event publishing framework converts the Openstack notifications to events format[4].
   Event publishing framework also has the ability to extract only some of the 'payload' data into
   a flat set of key-value pairs called 'traits' and publish the normalized 'event' with 'traits'
   extracted from the payload using a custom publisher.

   Extraction of certain fields into traits from the payload is
   driven by configuration file, but by default "publisher_id",
   'request_id', 'tenant_id', 'user_id' and 'project_id'
   fields are always extracted and added as 'traits'.

   The event can also have an optional field called 'raw' which has original
   notification, provided 'store_raw' option is set in ceilometer.conf

   Questions/TODO:

   * Q1: Does the store_raw field only apply to events, or to all notifications processed by
     Ceilometer?
   * We will have to find it out if it has any adverse impact on sample publisher. Though in the
     case of samples, monasca sample publisher definitely does not submit raw payload, so it must
     be getting dropped.

   Example Ceilometer Event data format:

   .. code-block:: javascript

      {
        "event_type": "compute.instance.create.start",
        "message_id": "c6149ba1-34b3-4367-b8c2-b1d6f073742d",
        "generated": "2015-09-18 20:55:37.639023",
        "traits": {
           "publisher_id": "compute.ccp-compute0001-mgmt",
           "request_id": "req-5948338c-f223-4fd8-9249-8769f7a3e460",
           "tenant_id": "a4f77",
           "project_id": "a4f77",
           "user_id": "be396488c7034811a200a3cb1d103a28"
         },
         "raw": {  "_context_auth_token": "42630b3ea13242fcad20e0a92d0207f1",
                   "_context_domain": null,
                   ...
                   ...
                   "event_type": "compute.instance.create.start",
                   "message_id": "c6149ba1-34b3-4367-b8c2-b1d6f073742d",
                   "payload": {
                       "access_ip_v4": null,
                       "access_ip_v6": null,
                       "architecture": null,
                       "availability_zone": null,
                       "cell_name": "",
                       "created_at": "2015-09-18 20:55:25+00:00",
                       "deleted_at": "",
                       "disk_gb": 1,
                       "display_name": "testeee",
                       "ephemeral_gb": 0,
                       "host": null,
                       "hostname": "testeee",
                       "image_meta": {
                           "base_image_ref": "df0c8",
                           "container_format": "bare",
                           "disk_format": "qcow2",
                           "min_disk": "1",
                           "min_ram": "0"
                       },
                      "image_name": "glanceaaa3",
                      "image_ref_url": "http://192.168.245.5:9292/images/df0c8",
                      "instance_flavor_id": "1",
                      "instance_id": "abd2ef5c-0381-434a-8efc-d7b39b28a2b6",
                      "instance_type": "m1.tiny",
                      "instance_type_id": 4,
                      "kernel_id": "",
                      "launched_at": "",
                      "memory_mb": 512,
                      "metadata": {},
                      "node": null,
                      "os_type": null,
                      "progress": "",
                      "ramdisk_id": "",
                      "reservation_id": "r-1ghilddw",
                      "root_gb": 1,
                      "state": "building",
                      "state_description": "",
                      "tenant_id": "a4f77",
                      "terminated_at": "",
                      "user_id": "be396488c7034811a200a3cb1d103a28",
                      "vcpus": 1
                      }
                }
      }


#. Key-Value pairs that can be extracted from 'payload' in form of traits
   can be defined in events definitions file.

   For example the following events definitions yaml specifies that for
   all events which have a prefix of "compute.instance.*" then
   add  "user_id", "instance_id", and "instance_type_id" as traits,
   after extracting values from "payload.user_id", "payload.instance_id",
   and "payload.instance_type_id" respectively.

   .. code-block:: yaml

   ---
   - event_type: compute.instance.*

     traits: &instance_traits
      user_id:
        fields: payload.user_id
      instance_id:
        fields: payload.instance_id
      instance_type_id:
        type: int
        fields: payload.instance_type_id

   We are for now proposing not to use this feature, of defining traits for each event
   extracting since we have the ability to store entire payload, via
   Monasca Events API.

   We can certainly look at enabling this feature in the future if we run into trouble storing
   entire JSON "payload" in Elasticsearch. This is certainly a nifty way to trim the amount
   of data that will be stored.

#. The proposed new Custom Monasca Ceilometer event publisher will run within Ceilometer's
   Notification Agent component. It will leverage Ceilometer's data processing pipeline[3] which
   converts notifications to Ceilometer's event format.  At the end of its processing, Monasca
   Ceilometer event publisher will convert Ceilometer Event data into Monasca Event format[6] and
   publish the monasca event to Monasca Events API.

#. Monasca Events API allows a field called 'payload' which can be in an arbitrary
   nested JSON format. Monasca-Ceilometer event publisher will extract JSON field called
   'payload' from 'raw' (JSON path notation: 'raw.payload'), publish the payload from the
   original notification to Monasca Events API.

   Example Monasca Event Format:

   .. code-block:: javascript

        events: [
        {
          dimensions": {
                "service": "compute.ccp-compute0001-mgmt",
                "topic": "notification.sample",
                "hostname": "nova-compute:compute
          },
          event: {

                  "event_type": "compute.instance.create.start",

                  "payload": {
                       "access_ip_v4": null,
                       "access_ip_v6": null,
                       "architecture": null,
                       "availability_zone": null,
                       "cell_name": "",
                       "created_at": "2015-09-18 20:55:25+00:00",
                       "deleted_at": "",
                       "disk_gb": 1,
                       "display_name": "testeee",
                       "ephemeral_gb": 0,
                       "host": null,
                       "hostname": "testeee",
                       "image_meta": {
                           "base_image_ref": "df0c8",
                           "container_format": "bare",
                           "disk_format": "qcow2",
                           "min_disk": "1",
                           "min_ram": "0"
                       },
                      "image_name": "glanceaaa3",
                      "image_ref_url": "http://192.168.245.5:9292/images/df0c8",
                      "instance_flavor_id": "1",
                      "instance_id": "abd2ef5c-0381-434a-8efc-d7b39b28a2b6",
                      "instance_type": "m1.tiny",
                      "instance_type_id": 4,
                      "kernel_id": "",
                      "launched_at": "",
                      "memory_mb": 512,
                      "metadata": {},
                      "node": null,
                      "os_type": null,
                      "progress": "",
                      "ramdisk_id": "",
                      "reservation_id": "r-1ghilddw",
                      "root_gb": 1,
                      "state": "building",
                      "state_description": "",
                      "tenant_id": "a4f77",
                      "terminated_at": "",
                      "user_id": "be396488c7034811a200a3cb1d103a28",
                      "vcpus": 1
                      }
                 },
            publisher_id: "compute.ccp-compute0001-mgmt",
            priority: "INFO"
         }
        ]

#. If no traits are specified in events pipeline yaml configuration file for an event
   Ceilometer's data processing pipeline will add the following default traits:

   * service: (All notifications should have this) notification’s publisher
   * tenant_id
   * request_id
   * project_id
   * user_id

   Note: "service" is not the service that produced the event as in say "compute", "glance",
   "cinder" but rather notification RabbitMQ publisher that produced the event
   e.g. "compute.ccp-compute0001-mgmt" so is not very useful.

#. Ceilometer event data is converted to Monasca event data format before being published to Monasca
   Event API. Following fields in Monasca Event data are not available in current Ceilometer Event
   data format:

   * "service"
   * "dimensions.topic"
   * "event.priority"

   We are proposing removing these fields from Monasca Event format (will be done as a separate
   spec/implementation process) for the following reasons:

   "service": Currently Openstack notifications do not specify a service, that
   generated the notification in a consistent way. It might be possible to create an external
   mapping file which maps event name to a service but its hard to maintain such mapping over a
   period of time.

   "dimensions.topic": This field is not available in the source Openstack notification

   "event.priority": This field is not currently available in Ceilometer Event format. It is
   available in the source Openstack notification. Note: If we think this field can be useful we can
   propose adding it to the Ceilometer Event format.

#. Following new fields will be added to Monasca Event data as dimensions:

   * "dimensions.publisher_id": Identifier for the publisher that generated the event. Maps to
     "traits.publisher_id" in Ceilometer event data.
   * "dimensions.user_id": Identifier for user that generated the event. Maps to "traits.user_id" in
     Ceilometer event data.
   * "dimensions.project_id": Identifier of the project that generated the event. Maps to
     "traits.project_id" or "traits.tenant_id" in Ceilometer event data.

#. hostname is available in the event payload, but its location might differ from event to event. We
   can use Ceilometer's event definitions config to always add a trait called "hostname" to all
   events. e.g. for compute.instance.* will have a trait called "hostname", which grabs data from
   "payload.hostname"

   .. code-block:: yaml

   ---
   - event_type: compute.instance.*

     traits: &instance_traits
      user_id:
        fields: payload.hostname

#. The proposed new Monasca Ceilometer event publisher will have the ability to submit event
   data in a batch and at a configurable frequency (similar to current samples publisher). The
   event data will be published if the items in the current batch reach their maximum size
   (config setting) or if certain time interval has elapsed since the last publish
   (config setting). This will make sure that the batch does not get huge at the same time
   there is no significant delay in publishing of the events to Monasca Events API.

#. Monasca Ceilometer event publisher will use service credentials from ceilometer configuration
   file (in "[monasca]" section) to get keystone token.

   Example "[monasca]" section in ceilometer config file
   .. code-block:: text

   [monasca]
   service_auth_url = https://localhost:5000/v3
   service_password = secretpassword
   service_username = ceilometer
   service_interface = internalURL
   service_auth_type = password
   # service_project_id may also be used
   service_project_name = admin
   service_domain_name = Default
   service_region_name = RegionOne

   The publisher will then make a POST request to Monasca Events /v1.0/events REST api[8] to publish
   events to  Monasca Events API.  The URL for the instance of Monasca Events API will be configured
   in the Ceilometer 'events-pipeline.yaml' file.  This has the added advantage of allowing
   different events to be published differently (see Ceilometer pipeline documentation [10]).

#. "tenant_id" and "user_id" that the notification relates to are available in "payload" section
   of the notification, and these notifications are generated by each service itself.

   There is no additional "Openstack-operator-agent" like component or functionality required to
   fetch that data from the service and publish to monasca event api on behalf of the original
   tenant.
   Ceilometer publishing pipeline simply extracts these "tenant_id" and "user_id" fields from the
   "payload" and makes those fields available as "tenant_id" and "user_id" traits, which would then
   be mapped to "dimensions.project_id" and "dimensions.user_id" fields in monasca events format.

   In other words, original "tenant_id" and "user_id" values are available in
   the payload of the notification, and will make its way to "dimensions.tenant_id"
   and "dimensions.user_id" in Monasca Event.

   Questions/TODO:
   * Q: Do we need to do anything special to handle multi-tenancy in monasca-events api like being
   done for metrics[9] ? Would original user_id and tenant_id in "dimensions.user_id" and
   "dimensions.tenant_id" fields in dimensions serve this purpose?
   * Q: In Ceilometer V2 API (which has been deprecated and removed), when querying data the role
   "admin" could access data for all tenants, whereas a user with "ceilometer-admin" role could
   access only data for a particular tenant. Can we implement something like this for
   monasca-events api when querying for data?

#. Monasca Ceilometer event publisher will also retry submitting a batch, in case Monasca
   Events API is temporarily unavailable or down. The retry frequency, the number of retries
   and the number of items that can be in the retry batch will also be set via configuration.


Alternative Solutions
---------------------

#. Standalone monasca event agent which reads Openstack notifications published to RabbitMQ
   (on "notification" topic) and publishes them to Monasca Events API.
   Pro:
   * No dependency on Telemetry project.
   * May be simple to develop if leverage the oslo.messaging functionality.
   * Ceilometer has *deprecated* the events functionality in the Stein release. [13]
   Con:
   * Another agent to convince users to install on their systems.
   * Reinventing work already done in the Ceilometer agent.  The OpenStack community already uses Ceilometer and contributes updates when something fails.
   This alternate solution will be detailed in a separate spec, as it is likely
   the long term solution Monasca will need.

#. Openstack Panko [5] is a event storage and REST API for Ceilometer.
   Pro:
   * An 'official' subproject within Telemetry, so there is some community recognition.
   Con:
   * Its primary storage is in a relational database which has problems with scale.
   * It is not maintained actively and not ready for production. [11]
   * It will be deprecated eventually. [12]

Data model impact
-----------------

None

REST API impact
---------------

#. We are proposing to tweak the Monasca Event data format by removing and adding following
   fields as mentioned in "Proposed change" section above.

   Remove fields (JSON path notation): "service", "dimensions.topic",
   "dimensions.hostname" and "event.priority"

   Add fields (JSON path notation): "dimensions.publisher_id", "dimensions.user_id" and
   "dimensions.project_id"

   This change will have an impact on Monasca Events API.

Security impact
---------------

The proposed Monasca Ceilometer events publisher will collect and publish
Openstack event (notification) data to Monasca API. Openstack notification
data does not have any sensitive data like 'tokens'.
Notifications do contain 'user_id' and 'project_id' fields but do not
contain any Personally Identifiable Information (PII) for the user or
the project.


Other end user impact
---------------------

None.

Performance Impact
------------------

#. The number of notifications(events) generated by different services will depend on the capacity
   of the cloud along with the number of resources being created by the users.

   For example, if there was a large number of compute VM's being created or destroyed it could
   lead to a surge in number of notifications (events) that would have to be published.  Optimum
   configuration options related to say event batch size and event batch interval would have to be
   documented, to reduce any adverse affect on performance.

#. Monasca Ceilometer publisher runs within Ceilometer Notification Agent component and invoked as a
   last step in its data processing pipeline. It is an additional component that will have to to be
   deployed on all the controller nodes.  We will have to evaluate the performance impact of
   Ceilometer Notification Agent when publishing events to Monasca Events API.


Other deployer impact
---------------------

#. The proposed new Monasca-Ceilometer events publisher will introduce
   few new configuration options like
   * events api endpoint
   * events batch interval
   * events batch size
   * events retry interval

#. Monasca Ceilometer Events publisher will have to to be added to Ceilometer's
   "[ceilometer.event.publisher]" section  entry_points.txt

   For example:

   [ceilometer.event.publisher]
   monasca = ceilometer.publisher.monclient:MonascaEventsPublisher

#. As part of developing new Monasca Ceilometer Events publisher devstack plugin would be updated to
   add the above configuration changes.


Developer impact
----------------

#. The proposed change to Monasca Event Format will have an impact on existing Monasca Event API,
   since Monasca Event Format will have to be tweaked.  (See REST API Impact section above)


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  joadavis, aagate

Other contributors:
  <launchpad-id or None>


Work Items
----------

#. Implement new Monasca Ceilometer Events publisher.

#. Implement monasca-ceilometer devstack plugin changes to deploy
   new events publisher.

#. Implement unit tests for Events publisher.

#. Implement change to Monasca Event format in Monasca Events API.


Dependencies
============

#. Monasca Events API 1.0: https://storyboard.openstack.org/#!/story/2001654

#. Monasca Ceilometer project: https://github.com/openstack/monasca-ceilometer

#. Ceilometer Data processing and pipelines:
https://docs.openstack.org/ceilometer/pike/admin/telemetry-data-pipelines.html

Testing
=======

#. New Monasca Ceilometer Event publisher unit tests will be added, which can test publishing with
   various config options events batch size, events batch interval, handling retry when Monasca
   Event API is not available.

#. Adding tempest tests for Monasca Ceilometer events publisher could be looked at as part of
   separate effort.

   Please note that current Monasca Ceilometer samples publisher does not have tempest tests either
   so having tempest tests for both events and samples publisher could be considered in the future.

Documentation Impact
====================

#. New Monasca Events Publisher config options will be documented

   * events api endpoint
   * events batch interval
   * events batch size
   * events retry interval

#. Recommended values for each of the config options will also be documented based on the size of
   the cloud and resources for Cloud Operators.

References
==========

[1] Monasca Events API 1.0: https://storyboard.openstack.org/#!/story/2001654

[2] Monasca Ceilometer project: https://github.com/openstack/monasca-ceilometer

[3] Ceilometer Data processing and pipelines:
https://docs.openstack.org/ceilometer/pike/admin/telemetry-data-pipelines.html

[4] Ceilometer Events: https://docs.openstack.org/ceilometer/latest/admin/telemetry-events.html

[5] Openstack Panko: https://github.com/openstack/panko

[6] Monasca Event Format:
https://github.com/openstack/monasca-events-api/blob/master/doc/api-samples/v1/req_simple_event.json

[7] Ceilometer System Architecture Diagram:
https://docs.openstack.org/ceilometer/ocata/architecture.html

[8] Monasca Events POST v1.0 API:
https://github.com/openstack/monasca-events-api/blob/master/api-ref/source/events.inc

[9] Cross-Tenant Metric Submission:
https://github.com/openstack/monasca-agent/blob/master/docs/MonascaMetrics.md#cross-tenant-metric-submission

[10] Ceilometer pipeline yaml documentation:
https://docs.openstack.org/ceilometer/latest/admin/telemetry-data-pipelines.html

[11] No future for Panko or Aodh:
https://julien.danjou.info/lessons-from-openstack-telemetry-deflation/

[12] Ceilometer Events deprecated means Panko also deprecated:
http://eavesdrop.openstack.org/irclogs/%23openstack-telemetry/%23openstack-telemetry.2018-10-10.log.html

[13] Ceilometer Events marked as deprecated in Stein:
https://review.openstack.org/#/c/603336/