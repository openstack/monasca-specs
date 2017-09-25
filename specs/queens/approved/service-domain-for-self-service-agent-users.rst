..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================================
Service Domain for Self Service Agent Users
===========================================

https://storyboard.openstack.org/#!/story/2001214

In order to send metrics and logs to Monasca the Monasca agent currently needs
a Keystone user with a special Keystone role, usually `monasca-agent`. This is
fine for infrastructure monitoring of an OpenStack cloud where the person
interested in monitoring can usually create user accounts, too, and where these
accounts' credentials are stored on the cloud's compute nodes and controllers
which should be well protected against breaches. Storing such credentials on
public facing instances to be monitored by Monasca is a problem because these
instances (a) tend to be more exposed and (b) because an OpenStack user
creating instances usually cannot create special purpose user accounts. This
spec proposes a solution to these two problems.

Problem description
===================

Currently there are two ways to submit metrics and logs for a given OpenStack
project:

1) Cross tenant submission: this requires a user with a special role that can
   submit metrics or logs on behalf of any arbitrary project. This role is
   currently used by compute nodes to submit libvirt metrics for the projects
   their running instances reside in. These roles are controlled by the
   `security/delegate_authorized_roles` setting for monasca-api and the
   `roles_middleware/delegate_roles` setting for monasca-log-api.

2) Metric submission by a user in the project: this is the normal way. Any user
   with the designated Monasca agent roles (`monasca-agent` by default) in a
   project can submit metrics and logs for this project. These roles are
   controlled by the `security/agent_authorized_roles` setting for monasca-api
   and the `roles_middleware/agent_roles` setting for monasca-log-api.

Both options are bad from a security point of view. (1) is worse, because it
will allow submission of arbitrary bogus log entries/metric events for arbitray
projects. Because of this high abuse potential, and because it is currently
only implemented in monasca-agent's libvirt monitoring plugin it is unlikely to
be employed for instance monitoring, though. (2) comes with a slightly lower
but still problematic security risk:

If a user wants to monitor their instances, they need to pass the Keystone
credentials of an OpenStack user with the `monasca-agent` role into their
instances. Like any OpenStack user with any role in a project, this user can
access arbitrary OpenStack APIs and create/delete resources at will. While it
would be possible to add global deny rules for the `monasca-agent` role to
every other OpenStack service's `policy.json`, this is unlikely to be
implemented in practice. Consequently, the compromise of an instance and its
`monasca-agent` complication will usually leave an attacker with unrestricted
out-of-band access to its creating user's Keystone project. This allows it to
typically allows it to, for example

* Create, view and delete Nova instances
* Create, view and delete Neutron networks
* Create, view and delete Cinder volumes
* Create, view and delete Glance images

In addition to this security problem there is a usability issue as well. A
regular OpenStack user can neither create Keystone users, nor can they assign
the `monasca-agent` role to these users. Consequently any user requiring
Monasca monitoring for their instance needs to ask somebody with admin
privileges to create a user and assign the `monasca-agent` role to that user.
Consequently, self-service instance monitoring is not possible.

Use Cases
---------

The change proposed by this spec will improve the situation outlined above  as
follows:

* End Users will be able to acquire access credentials for their instances'
  metrics and log agents in a self-service manner.
* End Users' attack surface from compromised instances with Monasca agents will
  be reduced to submission of bogus metrics/logs for their project.

Proposed change
===============

This change takes inspiration from OpenStack Magnum, particularily the fix for
CVE-2016-7404[0]. Before this fix, Magnum would create Keystone trustee users
in a separate domain, with one or more of a cluster owner's roles delegated via
Keystone trusts. These user accounts would only get used for submitting
certificate sign requests to the Magnum API in most cases so they had similarly
generous permissions to the monasca agent users. The fix for `CVE-2016-7404`
reduced the use of trusts to only the scenarios where they were really needed:
in the default case these users do not get any trusts delegated, nor do they
have roles assigned, rendering them useless for most OpenStack APIs. The Magnum
API's policy rules for `certificate:create` and `certificate:sign` are the sole
exception from this rule: they allow access if the user exists in the Magnum
domain and the user's ID matches the recorded user ID for a given cluster.

For Monasca, this spec proposes an analogous Keystone domain for agent users,
just like Magnum's trustee user domain. Likewise, the Monasca API service would
get access to an admin account for this domain so it can create users inside
the domain. The final puzzle piece are two extensions to the Monasca metrics
and log APIs:

1) A monasca-api endpoint that allows end users to get these special agent
   users created for their project and to retrieve their credentials.

2) A modification to log and metrics submission endpoints for monasca-api and
   monasca-log-api that allows submission for the project associated with the
   agent user in question.

In the remainder of this section you will find a detailed description of how
this change affects various parts of monasca-api.

Alternatives
------------

Some things could be implemented in a different manner from the approach
outlined below:

* It would be possible to substitute the rather heavyweight Keystone
  domain for the agent users by a lightweight, automatically generated API key.
  This would make for leaner credentials, but it would allow authorization for
  monasca-api/monasca-log-api submission entirely without Keystone. This is
  less of a technical and more of a political issue. Also, monasca-client would
  need additional, homegrown code for authenticating with this API key which
  may introduce additional security bugs. On the whole this is probably a bad
  idea (while discussing this on IRC people were in favour of using Keystone as
  well).

* Instead of recording project association and permissions for agent users in
  the database one could encode it in the agent users' user names. This would
  be less than elegant, though. On the other hand, we would not need to create
  database tables/add database client code to monasca-log-api.

* The current approach has monasca-api handling creation/deletion of agent
  users for both metrics and log submission. It would be conceivable to
  implement independent user creation for both APIs, but this would add
  considerable implementation overhead for no little benefit.

Data model impact
-----------------

We will need to introduce a new table `agent_users` with the following fields:

* `id` (string): the agent user's keystone user UUID. Unique primary key.
* `creator_id` (string): the creating user's keystone user UUID
* `project_id` (string): the project the user can submit logs metrics for
* `submit_metrics` (boolean): optional flag specified upon creation. Defaults
                   to True if unspecified. Controls whether the user is allowed
                   to submit metrics
* `submit_logs` (boolean): optional flag specified upon Creation. Defaults to
                True if unspecified. Controls whether the user is allowed to
                submit logs.

For this to work, monasca-log-api will need a database client implementation
and the configuration options to go with that, of course. In order to reduce
code duplication, as much database handling code from monasca-api as possible
will be moved to monasca-common from where both monasca-api and monasca-log-api
can use it.

REST API impact
---------------

The REST API needs to be modified in 3 places:

1. There needs to be a facility for self-service agent user creation

2. monasca-api needs to grant or deny metric submission based on the project
   an agent user is associated with and whether it has its `submit_metrics`
   flag set to `True`.

3. monasca-api needs to grant or deny log submission based on the project
   an agent user is associated with and whether it has its `submit_metrics`
   flag set to `True`.

In the remainder of this section these API changes are described in detail.

.. _agent_user_api:

Agent User Handling
^^^^^^^^^^^^^^^^^^^

To be able to create, delete and list agent users, and retrieve agent users'
credentials this spec proposes the following extensions to monasca-api:

::

    POST /v2.0/agent_users

This request creates agent users. The request body must follow the following
JSON schema:

::

    {
    type: "map",
    required: "true",
    "mapping":  {
      "password": { "type": "int", "required": false },
      "submit_metrics": { "type": "boolean", "required": false },
      "submit_logs": { "type": "boolean", "required": false }
      }
    }

The parameters behave as follows:

* `password`: if this is set, the provided password will be used as the agent
              user's password. Otherwise, a randomly generated 40 character
              string will be used.
* `submit_metrics`: if this is set to `False`, this agent user will not be
                    allowed to submit metrics to monasca-api. This parameter is
                    optional. If it is omitted, the default is `True` and the
                    agent user will be allowed to send metrics.
* `submit_logs`: if this is set to `False`, this agent user will not be
                 allowed to send logs to monasca-log-api. This parameter is
                 optional. If it is omitted, the default is `True` and the
                 agent user will be allowed to send logs to monasca-log-api.

This request will

* Return `200` with a JSON formatted database record for the agent user in the
  body if the request is successful. In addition to the database record the
  response will contain a `password` field with the newly created user's
  password. This password will *not* be recorded in the database.

* Return `500` with an error message in the body if user creation fails.

* Return `401` for unauthenticated users or users without any roles.

::

    GET /v2.0/agent_users

This request lists agent users. It will

* Return `200 OK` and a JSON formatted list of agent user database records for
  the requester's project. For requesters with the `admin` role, agent users
  from all projects will be listed. The list may be empty.

* Return `401` for unauthenticated users or users without any roles.

::

    GET /v2.0/agent_users/<id>

This request retrieves the database record for an agent user. It will

* Return `200` with a  JSON formatted record for the agent user
  identified by the Keystone user ID `<id>` in the body, provided there is a
  record for `<id>` in the database and the requester is allowed to access it.
  The requester is allowed to access a record if the requester's project
  matches `project_id` or the requester fullfills the `oslo.policy` `is_admin`
  criterion.

* Return `404` if there is no record for the user or the requester is not
  allowed to access it.

* Return `401` for unauthenticated users or users without any roles.


Extended Policy Check for Metric Submission
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The request

::

    POST /v2.0/metrics

will need to be extended to not only check for the roles configured in
`security/agent_authorized_roles` but it will also need to verify if

1. The requesting user is in the designated agent user domain

2. If so, look the user up in the `agent_users` table and submit the metrics
   being sent for the agent user's recorded project.

3. If the lookup fails for some reason (e.g. for a manually created user in the
   agent user domain that does not have a record in the database), the request
   is treated as unauthorized.


Extended Policy Check for Log Submission
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following requests

::

    POST /v3.0/logs

...will need to be extended to not only check for the roles configured in
`roles_middleware/delegate_roles` but it will also need to verify if

1. The requesting user is in the designated agent user domain

2. If so, look the user up in the `agent_users` table and submit the logs being
   sent for the agent user's recorded project.

3. If the lookup fails for some reason (e.g. for a manually created user in the
   agent user domain that does not have an associated domain), the request is
   treated as unauthorized.


Client impact
-------------

`python-monascaclient` will need to be extended to implement the API operations
described in the :ref:`agent_user_api` section above.

`monasca-agent` will not need to be modified.

Configuration changes
---------------------

monasca-api will need configuration settings to provide it with admin settings
for the agent user domain.

monasca-log-api will need configuration settings to provide it with access to
the monasca-api database.

Both monasca-api and monasca-log-api will need configuration that allows an
operator to disable this feature as desired. Since it allows users to create
monitoring users, it should be disabled by default. Optionally, it should be
possible to configure a Keystone role required to create agent users. If this
role is left unspecified, any user should be able to create agent users)

Security impact
---------------

This feature introduces a mechanism that allows regular users to create
unprivileged special purpose user accounts in a dedicated Keystone domain. This
might not be desirable for every operator, hence the provisions for disabling
or restricting it in the previous section.

The special purpose users in question do not have Keystone roles and are
therefore unusable for most purposes. There is a precedent for this in Heat and
Magnum. The former creates such users as targets for a Keystone trust delegated
by a Heat stack's creating user. The latter creates such users as of the fix
for CVE-2016-7404[0] and uses them in the exact same manner as the one proposed
by this spec.

Other end user impact
---------------------

Users will be able to create dedicated monitoring/logging users in a
self-service manner, which is an improvement over the current situation (they
need to ask somebody with admin privileges to create users with the special
`monasca-agent` role).

Performance Impact
------------------

This change may add a small performance penalty due to adding a database lookup
for every metrics submission attempt. This shouldn't be too bad, though,
because every metrics submission attempt already requires Keystone token
validation, beside which one database lookup is pretty small. Nonetheless, this
feature can be disabled until a fix is found if it turns out to cause major
performance issues.

Other deployer impact
---------------------

N/A

Developer impact
----------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jgr-launchpad

Work Items
----------

1. Add common database code to monasca-common.

2. Modify monasca-api to use database code from monasca-common and remove its
   own database code.

3. Implement agent user CRUD operations in monasca-api

4. Extend monasca-api policy enforcement code to authorize agent users

5. Extend monasca-log-api policy enforcement code to authorize agent users

Dependencies
============



Testing
=======


Documentation Impact
====================

The self service creation of agent users will need to be documented.

The various newly introduced configuration settings will need to be documented.

Creating a domain for agent users will need to be documented.

References
==========

[0]  https://github.com/openstack/magnum/commit/2d4e617a529ea12ab5330f12631f44172a623a14

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Queens
     - Introduced
