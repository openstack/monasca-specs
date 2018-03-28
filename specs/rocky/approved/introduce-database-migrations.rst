..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================
Introduce Database Migrations
=============================

https://storyboard.openstack.org/#!/story/2001654

Most OpenStack services use `SQLAlchemy Migrate
<https://sqlalchemy-migrate.readthedocs.io/en/latest/>`_ or `Alembic
<http://www.alembic.io/>`_ to provide migration scripts that (a) update the
service's database schema if it changes and (b) migrate data as required.
Currently, Monasca does not follow this pattern. Instead, it provides simple
SQL scripts for PostgreSQL or MySQL databases that are only useful for
initializing a database schema. For any subsequent changes required by updated
code operators are on their own, i.e. they need to apply these manually to keep
Monasca working. This spec proposes introducing (a) database migrations and (b)
a separate tool that analyses an existing database's schema state and adds the
necessary metadata to update it through migrations in the future.


Problem description
===================

In a nutshell, the database schema used for holding various alarm,
notification, and metric metadata is not updateable. This is due to the source
of truth for the schema being a simple SQL script that is only useful for
schema initialization on a blank database.

The group most affected by this is operators, who are faced with several
unattractive choices when a code change requires a database schema change:

#. They can drop their whole Monasca database and recreate it from scratch with
   the updated SQL script. That way they will lose all of their alarm
   definitions along with notification settings, their complete alarm history,
   and some metrics metadata.

#. They can manually update the database and migrate data (in the case of
   columns/tables getting renamed). This is error prone and carries a risk of
   data loss.

#. They can decide to live without that code change. If the change breaks on an
   outdated database they may even have to create and maintain code patches for
   that.

Besides operators, the lack of database migration infrastructure is a
significant obstacle to Monasca developers as well: it turns any data model
change into a breaking change due to the schema update that requires. Thus the
current state of affairs prevents new features that require database
changes or renders them extremely unpopular at the very least.

Use Cases
---------

#. Operators will be able to update Monasca without risking breakage due to
   database changes.

#. Developers can modify the data model without breaking existing
   installations.

Proposed change
===============

The proposal aims to improve the current state of affairs by adding Alembic
based command line tooling for the following operations:

#. A legacy database migration command line tool. This command line tool will
   detect which revision of the database schema SQL script was used based on
   the tables and columns currently in the database. It will then initialize
   the database's migration meta data to allow for regular database migration
   from that point forward.

#. A database migration command line tool with multiple base revisions to
   cover the following scenarios:

     #. An uninitialized database
     #. A schema state resulting from any of the existing SQL script
         revisions.

Both (1) and (2) may be implemented as part of the same tool.

A database schema based on third party SQL scripts (which may be found in
various configuration management tools used for deploying Monasca) is not
supported.

Alternatives
------------

This change is long overdue and best practice throughout OpenStack. That being
said, it would be possible to considerably reduce its scope by omitting the
heuristics for detecting an existing legacy database schema. That way operators
would be forced to drop the database and recreate it using migrations. If
possible we should not impose this on operators.

It would also be possible to use plain SQLAlchemy Migrate rather than Alembic
to implement migrations, but OpenStack appears to have standardized on Alembic
by now (see https://wiki.openstack.org/wiki/OpenStack_and_SQLAlchemy for a
discussion of why that happened).

Data model impact
-----------------

There will be no impact on the data model itself. There will only be changes to
how the data model is synchronized to the database's run time schema.

REST API impact
---------------

There is no REST API impact.

Security impact
---------------

The only slightly security related aspect of this change is providing the
database migration command line tools with access credentials for the data
base. The accepted best practice for this is running it as root on the machine
where the API service (in our case `monasca-api`) is also running and retrieve
these credentials from the API service's configuration.

Other end user impact
---------------------

N/A

Performance Impact
------------------

N/A

Other deployer impact
---------------------

Configuration management that uses the legacy SQL scripts must be changed to
use the new migration tools. Among other things, the Monasca devstack plugin
must be modified to use them. Users updating to a Monasca version with this
feature may need to run a special migration to get versioning metadata added to
their database schema.

Developer impact
----------------

Any developer making data model changes after this feature is implemented will
have to write a migration for updating the database schema accordingly.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <jgr-launchpad>

Tasks
-----

* Add command line tool(s) for regular migration (i.e. migration of a versioned
  database) and transition of an unversioned database to a versioned one.

* Create migration chains from:

  * Empty database

  * All revisions of the schema file in the repository.

Dependencies
============

N/A

Testing
=======

Any meaningful testing can only take place in a full Monasca deployment with an
actual database to test against. In particular, testing attention should focus
on testing this with all revisions of the legacy SQL script in
`devstack/files/schema/mon_mysql.sql`. This testing can take place at an early
stage during Devstack setup as follows:

#. Each SQL script revision is checked out from the git repository, applied to
   the database and converted to a "migratable" database by running the
   conversion operation on the database. After each iteration, the database is
   dropped to prepare for the next revision.
#. As a final step, the initial migration for a blank database is performed and
   Devstack proceeds normally.

Since PostgreSQL is deprecated in OpenStack it will be sufficient to implement
testing with the MySQL flavoured SQL script.

Documentation Impact
====================

This feature needs to be documented in operator/deployer documentation to
ensure operators use migrations rather than legacy SQL scripts.

References
==========

* Etherpad from Rocky PTG where this was discussed:
  https://etherpad.openstack.org/p/monasca-ptg-rocky
