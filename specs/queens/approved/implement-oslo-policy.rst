..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================
Use oslo.policy for Monasca APIs
================================

https://storyboard.openstack.org/#!/story/2001233

Presently, neither monasca-api nor monasca-log-api use `oslo.policy`. Instead,
they contain their own policy enforcement code. Without `oslo.policy` they will
not be able to implement the policy in code community goal for Queens[0], which
requires the use of oslo mechanisms for defining and enforcing the policy.

Problem description
===================

Since neither monasca-api nor monasca-log-api use oslo.policy for policy
enforcement, they cannot describe their policy in code using `oslo.policy`
mechanisms as mandated by the community goal[0].

Use Cases
---------

The change proposed by this spec will improve the situation outlined above  as
follows:

#. Policy will be enforced in the OpenStack wide standard manner by
   `oslo.policy`. This reduces the maintenance burden on Monasca developers
   because they will no longer need to maintain Monasca's own policy enforcement
   code.

#. All default policy rules will be available in a standard format to all
   interested parties, which fulfils the community goal.

#. Operators will be able to override the Monasca API and Monasca Log API
   default policies using `policy.json` files and run command line tools to
   generate a full `policy.json` for either service (e.g. for auditing
   purposes).

#. The ability to use new tooling in oslo.policy that let's developers
   deprecate or change policy defaults in a way operators can consume.

Proposed change
===============

This spec proposes the following changes to `monasca-common`, `monasca-api`,
`monasca-log-api` and the upcoming `monasca-events-api`:

#. Implement a policy enforcement engine in `monasca_common.policy`. We can
   probably model this on Nova's policy enforcement engine[1]. We will have to
   modify it somewhat to account for the fact that we have multiple APIs that
   use the same policy engine.

#. Define a module that contains the default policy rules in both `monasca-api`
   and `monasca-log-api` and exposes them to the enforcement engine (in
   `monasca-events-api` there is such a module already). Nova's approach of
   having a `list_rules()` method[2] should work just fine for us.
   We can either copy Nova's approach of aggregating individual endpoints'
   policies in that central module[3] or just have them in there directly.

#. Modify existing policy enforcement code in `monasca-api`, `monasca-log-api`
   and `monasca-events-api` by code to use the enforcement engine from
   `monasca-common`.

#. Add monasca-api-policy and monasca-log-api-policy command line entry points
   that allow the user to generate a policy.json file for both APIs.

Alternatives
------------

None, since this is a community goal. One thing that could be done differently
is the policy enforcement engine: it would be conceivable to have independent
enforcer implementations in both monasca-api and monasca-log-api. This would
needlessly violate the DRY principle, though.

Data model impact
-----------------

This change does not impact the data model.

REST API impact
---------------

This change must be implemented in a way that creates no discernible impact for
the API's consumers.

Client impact
-------------

N/A

Configuration changes
---------------------

This change will continue to use the same `monasca-api` and `monasca-log-api`
configuration settings we already use for policy enforcement:

* `agent_authorized_roles`, `default_authorized_roles`,
  `delegate_authorized_roles` and `read_only_authorized_roles` for `monasca-api`

* `default_roles`, `agent_roles` and `delegate_roles` for `monasca-log-api`

The only difference is that a different implementation
(`monasca-common.policy`) will use them now.

Additionally, the policy enforcer will allow operators to create a
`policy.json` file for each API service that overrides the in-code defaults.

Security impact
---------------

This change introduces a way for operators to influence monasca-api and
monasca-log-api policy through configuration files. If they misconfigure policy
that way, they may allow unauthorized users to send or retrieve metrics and
logs.

Other end user impact
---------------------

N/A

Performance Impact
------------------

N/A

Other deployer impact
---------------------

N/A

Developer impact
----------------

Developers that introduce new API operations will need to register policy rules
for these endpoints once this feature is implemented.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  amofakhar

Work Items
----------

#. Add policy enforcement module to `monasca-common`. It might be a good idea
   to have an extension mechanism for the policy's target analogous to this
   one[1] in Magnum to account for the different role variables from the
   configuration file needed by `monasca-api` and `monasca-log-api`,
   respectively. That way a generic policy enforcer can be made flexible to the
   point where all the role differences between `monasca-api` and
   `monasca-log-api` can be expressed in terms of policy rules.

#. Retire policy enforcement code in `monasca-events-api` in favour of the
   policy enforcement module in `monasca-common`.

#. Add policy registration code to `monasca-api`.

#. Use policy enforcement module in `monasca_api.v2.reference.helpers.validate_authorization()`

#. Add policy registration code to `monasca-log-api`

#. Remove policy enforcement code from `monasca_log_api.middleware.role_middleware.`
   (for validating agent role) and `monasca_log_api.app.base.request`
   and replace it by using `monasca-common.policy` from either
   `monasca_log_api.middleware.role_middleware.` or
   `monasca_log_api.app.base_request` (to have centralized enforcement in one
   spot).

#. Add `monasca-api-policy` console script to `monasca-api`

#. Add `monasca-log-api-policy` console script to `monasca-log-api`


Dependencies
============

N/A

Testing
=======

Existing policy enforcement tests are likely to need a major overhaul.

Documentation Impact
====================

The following things need to be documented for operators:

#. Their newly added ability to create `policy.json` files for Monasca API services

#. The functionality of the `monasca-api-policy` and `monasca-log-api-policy` scripts

References
==========

[0] https://governance.openstack.org/tc/goals/queens/policy-in-code.html

[1] https://github.com/openstack/nova/blob/master/nova/policy.py

[2] https://github.com/openstack/nova/blob/master/nova/policy.py#L207

[3] https://github.com/openstack/nova/blob/master/nova/policies/__init__.py

[2] https://github.com/openstack/magnum/blob/master/magnum/common/policy.py#L102

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Queens
     - Introduced
