.. _xena-priorities:

===========================
Xena Project Priorities
===========================

List of priorities the Monasca drivers team is prioritizing in Xena.

The owners listed are responsible for tracking the status of that work and
helping get that work done. They are not the only contributors to this work,
and not necessarily doing most of the coding!

The implementation progress on these priorities and other identified important
tasks is tracked in `this board`_.

.. _this board: https://storyboard.openstack.org/#!/board/236

Essential Priorities
~~~~~~~~~~~~~~~~~~~~

+----------------------------------------------------+---------------------------+
| Title                                              | Owners                    |
+====================================================+===========================+
| `Migrate CI/CD from Travis-CI to Github actions`   | chaconpiza                |
+----------------------------------------------------+---------------------------+
| `Update Docker Images`                             | chaconpiza                |
+----------------------------------------------------+---------------------------+
| `Update https://github.com/monasca/monasca-docker` | chaconpiza                |
+----------------------------------------------------+---------------------------+


High Priorities
~~~~~~~~~~~~~~~

+----------------------------------------------------+-------------------------+
| Title                                              | Owners                  |
+====================================================+=========================+
| `Thresholding engine in cluster mode`              | chaconpiza              |
+----------------------------------------------------+-------------------------+
| `Add Time and Times in Monasca UI`                 |                         |
+----------------------------------------------------+-------------------------+
| `Define Prometheus based architecture`             |                         |
+----------------------------------------------------+-------------------------+
| `Application Credentials`_                         |                         |
+----------------------------------------------------+-------------------------+
| `Middleware upgrades ELK 7.3.0 -> OpenDistro`      |                         |
+----------------------------------------------------+-------------------------+


Optional Priorities
~~~~~~~~~~~~~~~~~~~

+------------------------------------------+-------------------------+
| Title                                    | Owners                  |
+==========================================+=========================+
+------------------------------------------+-------------------------+
| `Selenium Tests for Monasca-UI`          |                         |
+------------------------------------------+-------------------------+
| `OpenStack Client Integration`           |                         |
+------------------------------------------+-------------------------+

Details
~~~~~~~

Application Credentials
-----------------------

`Keystone appliction credentials <https://docs.openstack
.org/keystone/latest/user/application_credentials.html>`_ offer the mechanism
to allow applications to authenticate to Keystone. The ability to specify
`access rules <http://specs.openstack
.org/openstack/keystone-specs/specs/keystone/stein/capabilities-app-creds
.html>`_ for application credentials has been implemented in the Train cycle.

The goal of this story is to add application credentials support in
*monasca-agent*. This will prevent the security risk of revealing OpenStack
user's password when installing the agent on the tenants environment. The
access rules of these application credentials should be limited to posting
measurements. *monasca-setup* should be extended to automatically generate such
credentials and save them in configuration file if needed.

Similar task should be implemented in *monasca-grafana-datasource*.

Stories:

* https://storyboard.openstack.org/#!/story/2005622
* https://storyboard.openstack.org/#!/story/2005623

