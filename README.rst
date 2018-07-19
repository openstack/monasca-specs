========================
Team and repository tags
========================

.. image:: https://governance.openstack.org/tc/badges/monasca-specs.svg
    :target: https://governance.openstack.org/tc/reference/tags/index.html

 .. Change things from this point on

======
README
======

Monasca Specifications
======================


This git repository is used to hold priorities and approved design
specifications for additions to the Monasca project. Reviews of the specs are
done in gerrit, using a similar workflow to how we review and merge changes to
the code itself.

The layout of this repository is::

  priorities/<release>/
  specs/<release>/

Where there are three sub-directories in ``specs``:

specs/<release>/approved
  Specifications approved, but not yet implemented

specs/<release>/implemented
  Implemented specifications

specs/<release>/not-implemented
  Specifications that were approved but are not expected to be implemented.
  These are kept for historical reference.
