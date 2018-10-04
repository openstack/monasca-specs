..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================
Merge Monasca APIs
==================

https://storyboard.openstack.org/#!/story/2003881

Monasca currently has 3 APIs; one for metrics, one for logs and one for
events. Historically, this has created additional overhead for developers
upgrading or adding new features to the APIs, for system administrators
configuring the APIs, and for packagers generating Monasca distributions.
The purpose of this spec is to consider whether the Monasca APIs should
be merged into a single, unified API.

Problem description
===================

The Monasca APIs are written in Python and share a lot of functionality,
some, but not all of which has been factored out to the Monasca common
library. Here lies the first problem:

1. There exists significant technical debt.

There is a significant amount of common functionality that has not been
factored out to Monasca common. For example, the Monasca API repo contains
code for validating metrics and so does the Monasca common library. The
same is true for query authorisation, where validation code lives in
both repos. This technical debt could of course be addressed, but how
did it come about?

2. Updating the APIs to support common features incurs significant
   developer and reviewer overhead compared to updating a single API.

The APIs are all written slightly differently. Adding support for
Oslo Policy or uWSGI to one API is not the same as adding it to another
API. Furthermore, changes frequently have to be made across multiple
repos. Whilst Zuul is well suited to coordinating this task, it requires
careful synchronisation of versions by developers and meticulous
attention from reviewers. If one modifies code which is common to all
three APIs, they must systematically verify that in each case the changes
work as intended. Of course, automated testing can provide relief here, but
it doesn't make the burden disappear entirely.

3. Historically it has been difficult to maintain a standard experience
   across multiple APIs.

At times, APIs have diverged when in an ideal world they should not have.
For example, when adding support for a common feature, work has traditionally
been focused on one API to keep the task simple, with the view to adding
support for the other APIs at a later date. If works stops, for example due
to a release, or due to a developer moving on, the APIs can be left in a
diverged state for a significant amount of time. Of course, one solution
is to block merging of a common feature until the work is complete
in all three APIs, and all common code is added to the Monasca
common library. In practice, this has been prevented by the number of man
hours available. For example, a contributor may only be interested in
metrics. They may not be able to justify spending time working on the other
APIs.

4. Packaging, deploying, and configuring 3 APIs is more complex than
   deploying a single, unified API.

This is an obvious point, but it can be made worse by 3). For example,
historically, it has not always been possible to run all APIs in the same
way. Configuration of common functionality such as Kafka has not
always been uniform. There is extra build system configuration,
additional keystone endpoints that need to be registered and monitoring
the availability of 3 APIs is more difficult than one.

Use Cases
---------

As a developer I would save time by implementing a feature common
to all APIs in a single repo, rather than across four repos.

As a reviewer I will save time by not having to think about how
a change may impact multiple repos.

As a deployer I will save time by having two fewer services to deploy
and configure.

As a security analyst I can focus my efforts reviewing a single API,
rather than three.

As a user of Monasca I would like a consistent experience, including usability
and documentation.

As a packager I would like to have a single Monasca API, rather than
three to save time configuring and maintaining my build system.

Proposed change
===============

Merge all APIs into a single unified API. Merge all common API code from the
Monasca common repo into the unified API repository. Specifically, it is
proposed to merge all relevant code into the Monasca API.

Alternatives
------------

1. Refactor all APIs so that the code is standard. Prevent merging of common
   features until the work is complete across all APIs, and all common code
   has been factored out into the Monasca common library. From historical
   experience this will be difficult without additional developers.

2. Don't do anything and carry on working around the technical debt. In the
   long term this is likely to make it more difficult to add new features, and
   require more time for maintenance.

Data model impact
-----------------

None

REST API impact
---------------

Aside from the fact that a single service will implement the combined schema
from all APIs, the calls should not change. We should be careful when merging,
for example dimension validation code, that we do not break things which were
accepted in one API, but not another. An ideal result would be that we use
the same code for parsing common fields such as dimensions for all API calls.

Security impact
---------------

If a single API was previously exposed publicly, deploying the unified API
will increase the surface area for attack. However, in general, it will be
easier to review changes to make security improvements due to the reduced
developer and reviewer overhead.

During the deprecation period of the Events and Log API, security fixes made
to the unified API may need to be backported.

Other end user impact
---------------------

All services talking to the Monasca Events and Log APIs will need
to be directed at the unified API. For services which use Keystone
to lookup the Monasca endpoints this should occur automatically. Other
services such as the Fluentd Monasca output plugin will need to be
reconfigured manually. A grace period where the Monasca Events and Log
APIs are still supported is one possibility to make this transition
easier.

Performance Impact
------------------

No direct impact.

In general, it will be less effort to profile and optimise a single API than
it would be to do the same for all three APIs.

In clustered deployments with specialised nodes (for example a dedicated
logging node, hosting the Monasca Log API) the unified API can be deployed
as a direct replacement, and the additional functionality can simply be
ignored.

Other deployer impact
---------------------

For Monasca users supporting legacy releases, any security or bug fixes
made to the unified API may need to be backported to the individual APIs.

All actively maintained deployment solutions will need to be updated to
deploy the unified API. For example, Monasca-Docker, DevStack, Kolla,
OpenStack Ansible, and Helm.

In the case of DevStack we should merge the three existing plugins into
one. The resulting plugin should have options like `log_pipeline_enabled`
and `metrics_pipeline_enabled` to support enabling those pipelines
separately. This is useful, for example, when DevStack is used in OpenStack
CI to allow testing changes localised to specific areas more efficiently.

Developer impact
----------------

The motiviation behind this change is to reduce the burden placed on
developers and reviewers when making improvements to the Monasca APIs. It
is hoped that this will lead to an increase in developer productivity.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Work Items
----------

* Review test coverage of APIs, and add coverage for any missing areas.
* Review test coverage of Monasca common and add coverage for any missing areas.
* Merge Monasca common Python code into the Monasca API. Common functionality
  should include:

  * authorisation
  * validation
  * OpenStack policy
  * WSGI deployment

* Implement Log API schema in the Monasca API and port tests.
* Implement Events API schema and port tests.
* Merge DevStack plugins into a single plugin and add support for enabling
  pipelines individually.
* Deprecate Monasca Log and Event APIs.
* Merge and update documentation

Dependencies
============

No additional dependencies are added. The dependency on Monasca common can
be removed.

Testing
=======

The Monasca API and Log API Tempest test plugins have already been merged into
one plugin. Any Tempest tests which exist for the Events API should also be
merged into the unified plugin. The Tempest plugin will need to be updated to
use the unified API.

Unit tests from the Log API and Events API repo will need to be ported to the
Monasca API (unified) repo. Some of these tests may be redundant.

Documentation Impact
====================

Three sets of documentation will be reduced to one. Whilst it will take
some effort to merge the documentation, it should hopefully be more
consistent.

References
==========

None

History
=======

None
