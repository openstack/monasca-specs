..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Monasca services in Docker
==========================

The main task of this story is to move building and publishing Docker images
of all Monasca components from https://github.com/monasca/monasca-docker
to their respective OpenStack repositories.

Problem description
===================

At the moment Docker images for Monasca components are built from Dockerfiles
provided in `monasca-docker repository <https://github.com/monasca/monasca-docker>`_.
The process of building and publishing the images has to be explicitly
triggered and is described
`here <https://github.com/monasca/monasca-docker/blob/master/CONTRIBUTING.md#releasing-changes>`_.
Due to the separation of image definitions and the actual upstream code they
can easily diverge.

Use Cases
---------

Have supported and standardized option to deploy Monasca services with Docker.

From the development point of view very little will change.

A developer is writing code and putting it into Gerrit, then automatic process
test if Docker image will be built properly.

The images can be used in integration tests running in OpenStack CI.

End User should see no change.

Deployer has one more supported way of deploying Monasca.


Proposed change
===============

Every repository with each component would have additional `docker` folder
that would contain all necessary files for building Docker image.

Example files:

* `Dockerfile`
* `README`
* Starting script that will template needed configuration files, wait for other
  needed components to be available (like Keystone) and start service
  on Docker image run.
* Templates for configuration files, bootstrap scripts for configuring database
  schemas etc..

The process should be started after commit land in git repository (so all
tests passing).
The goal is a fully automatic process without the need for human intervention.
Every image should have an easy and standardized way of getting information
from what version of Monasca component it was built (definitely needed for
`master` and `latest` tags).
Every Dockerfile will be in a separate repository so there is a need
for standardization of these files so that they look mostly the same.
There are linters for Dockerfiles, one need to be chosen and added to
the testing process (will help with standardization of the Dockerfiles,
check Kolla jobs).

Images will be build with Python 3.5. If any Monasca service does not support
Python 3, we will wait for support before creating container for it.

Images will be published to https://hub.docker.com/u/monasca/

Each image should have README file in RST format with information about running
specific component.

We will be supporting Docker actual stable version and 2 previous versions.

All stable branches will have specific versions of Docker that they are build
with and this versions will be frozen at the moment of branching them from
master. Master branch will be build with 3 newest Docker versions.

All images will be based on custom `monasca-base` image that is using official
Python on Alpine Linux to conserve disc space. We will be using ideas for
minimal configuration from other base image examples (like based on Ubuntu
https://github.com/phusion/baseimage-docker or based on Alpine
https://github.com/blacklabelops/baseimages).

All images should be created in a way that will try to make them as small
as possible without compromising on functionality. Possible ways of making them
smaller could be found in articles like the following

https://blog.codeship.com/alpine-based-docker-images-make-difference-real-world-apps/

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

All images standardize on Jinja2 for templating configuration files with the
following tool:

  * https://github.com/Aisbergg/python-templer - Python3 command-line tool for
    templating in shell-scripts, leveraging the Jinja2 library

    * Allows to use environment variables
    * YAML data sources supported
    * LGPL license but we are not linking against it, only using CLI


Alternatives
------------

As an example of the CI configuration, we could use Kolla process
of creating Docker images:
https://github.com/openstack/kolla
https://hub.docker.com/u/kolla/
It is though much too complex for what we need, so should be used just
as a reference.

For templating:

  * https://github.com/wrouesnel/p2cli - command line tool for rendering pongo2
    (Django-syntax like, similar to Jinja2) templates (Go binary, 4x smaller
    size if the image does not already have Python)

    * Allows to use environment variables
    * YAML, JSON data sources supported

  * https://github.com/jwilder/dockerize - Utility to simplify running
    applications in docker containers (Go binary)

    * Generate application configuration files at container startup time from
      templates (Go text/template) and container environment variables
    * Tail multiple log files to stdout and/or stderr
    * Wait for other services to be available using TCP, HTTP(S), unix before
      starting the main process.

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

All services will be closed in Docker container.

Other end user impact
---------------------

None

Performance Impact
------------------

Because of additional Docker layer services could run slower.

Other deployer impact
---------------------

Deployment should be easier as deployer would need to create configuration
and services with all dependencies will be enclosed in Docker images.

Developer impact
----------------

Features adding new dependencies or changing the way Monasca components
are installed would have to be reflected in Docker image definitions.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  dobroslaw-zybort <dobroslaw.zybort@ts.fujitsu.com>

Work Items
----------

What is needed:

* Building test images for every change in Gerrit.

  * Automated process of building images on every git commit and pushing them
    to the Docker Hub.
  * Automated process of building images on every tag and pushing them
    to the Docker Hub.
  * Automated process of building images on OpenStack releases and pushing
    them to the Docker Hub (could be the second step).

* Adding labels with source code base information of the image (e.g.
  build-date, commit-sha1).

* Running integration tests on each code commit to verify the Docker binary.
  Single node deployment setup with Docker Compose.

  * Tempest tests.
  * Smoke tests.

Five types of tags on Docker Hub:

* On every new release/tag.
* `latest` pointing to the last tag.
* `master` pointing to the last git commit - useful for testing last.
  changes/fixes before release.
* Tags for the last commit on stable OpenStack versions (`queens`, `rocky` etc.) -
  useful for testing last changes/fixes before release.

Optional:

* Build all images with Python 3.6 and test stability.

* Run multi node deployment integration tests with Kubernetes Helm.

Needs to remember (to make maintenance easier):

* Using proper init process.
* Cleaning after the installation process.
* Same name of starting script in all repositories.
* Use same config templating mechanism in all repositories.
* All components should be downloaded from Git repository and have a possibility
  to change branch for building Docker image (useful for testing changes
  proposed on Gerrit).


Non objectives of this story (could be next steps):

* Automated process of building images on specific past commits and pushing
  them to the Docker Hub.
* Migrate Devstack to Docker images of Monasca.


Dependencies
============

What should be moved to where:

* https://github.com/monasca/monasca-docker/tree/master/monasca-agent-base =>
  https://github.com/openstack/monasca-agent
* https://github.com/monasca/monasca-docker/tree/master/monasca-agent-collector =>
  https://github.com/openstack/monasca-agent
* https://github.com/monasca/monasca-docker/tree/master/monasca-agent-forwarder =>
  https://github.com/openstack/monasca-agent
* https://github.com/monasca/monasca-docker/tree/master/monasca-api-python =>
  https://github.com/openstack/monasca-api
* https://github.com/monasca/monasca-docker/tree/master/monasca-client =>
  https://github.com/openstack/python-monascaclient
* https://github.com/monasca/monasca-docker/tree/master/monasca-log-api =>
  https://github.com/openstack/monasca-log-api
* https://github.com/monasca-docker/monasca-notification =>
  https://github.com/openstack/monasca-notification
* https://github.com/monasca/monasca-docker/tree/master/monasca-persister-python =>
  https://github.com/openstack/monasca-persister
* https://github.com/monasca/monasca-docker/tree/master/monasca-python =>
  https://github.com/openstack/monasca-common
* https://github.com/monasca/monasca-docker/tree/master/monasca-thresh =>
  https://github.com/openstack/monasca-thresh

  * will need also https://github.com/monasca/monasca-docker/tree/master/storm =>
    https://github.com/openstack/monasca-thresh

* https://github.com/monasca/monasca-docker/tree/master/tempest-tests =>
  https://github.com/openstack/monasca-tempest-plugin

Not in scope:

* https://github.com/monasca/monasca-docker/monasca-alarms - This image
  contains a container that can be used to create Monasca Notifications
  and Alarm Definitions.
* https://github.com/monasca/monasca-docker/monasca-log-agent - Logstash
  output monasca_log_api plugin.
* https://github.com/monasca/monasca-docker/monasca-log-metrics - Contains
  Logstash configuration to transform logs into metrics based on log's severity.
* https://github.com/monasca/monasca-docker/monasca-log-persister - Contains
  Logstash configuration to save logs inside **log-db** (i.e. ElasticSearch).
* https://github.com/monasca/monasca-docker/monasca-log-transformer - Image
  contains Logstash configuration to detect log's severity.

Testing
=======

At this moment CI for https://github.com/monasca/monasca-docker run two types
of tests on each change:

* tempest-tests https://github.com/monasca/monasca-docker/tree/master/tempest-tests
* smoke-tests https://github.com/monasca/monasca-docker/tree/master/smoke-tests

  * https://github.com/monasca/smoke-test


Both of them, at the moment, are running on metrics part of the Monasca stack.

Tests should consider if tested service started and is behaving correctly
in built and running Docker container. We need to decide if we want to run
all tests on the whole Monasca stack on every change or if we should create
some smaller tests for each separate service.

Documentation Impact
====================

Basic installation instructions should be added here [1] and published
to https://docs.openstack.org

[1] https://git.openstack.org/cgit/openstack/monasca-api/tree/doc/source/install

Now high level documentation is stored in:
https://github.com/monasca/monasca-docker/tree/master/docs

Separate images also have `README.md` files that give lower level information.

Documentation should contain all necessary information how to configure and run
all services.

References
==========

* https://github.com/monasca/monasca-docker

* http://eavesdrop.openstack.org/meetings/monasca/2018/monasca.2018-01-10-15.00.log.html


History
=======

Optional section intended to be used each time the spec is updated to describe
the new design, API or any database schema updated. Useful to let reader
understand what's happened at the time.

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Rocky
     - Introduced
