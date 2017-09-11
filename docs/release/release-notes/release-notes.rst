.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

================================
OPNFV IPv6 Project Release Notes
================================

This document provides the release notes for Euphrates of IPv6 Project.

.. contents::
   :depth: 3
   :local:


Version History
---------------

+--------------------+--------------------+--------------------+--------------------+
| **Date**           | **Version**        | **Author**         | **Comment**        |
|                    |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| 2017-09-10         | 0.1.0              | Bin Hu             | First draft        |
|                    |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
|                    | 0.x.0              |                    | Baseline draft     |
|                    |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
|                    | 1.0.0              |                    | Release Ready      |
|                    |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+

Important Notes
---------------

**Attention:** Please be aware that:

* Since Danube, Apex Installer no longer supports Open Daylight L2-only
  environment or ``odl-ovsdb-openstack``. Instead, it supports Open Daylight L3
  deployment with ``odl-netvirt-openstack``.
* IPv6 features are not fully supported in Open Daylight L3 with
  ``odl-netvirt-openstack`` yet. It is still a work in progress.
* Thus we cannot realize Service VM as an IPv6 vRouter using Apex Installer
  under OpenStack + Open Daylight L3 with ``odl-netvirt-openstack environment``.

For details, please refer to our `User Guide <../userguide/index.html>`_.

Summary
-------

This is the Euphrates release of the IPv6 feature as part of OPNFV, including:

* Installation of OPNFV on IPv6-Only Infrastructure by Apex Installer
* Configuration of setting up a Service VM as an IPv6 vRouter
* User Guide, which analyzes the gap of IPv6 support in OpenStack Ocata
  and OpenDaylight Carbon.

Please refer to our:

* `Installation Guide <../installation/index.html>`_
* `Configuration Guide <../configguide/index.html>`_
* `User Guide <../userguide/index.html>`_

Known Limitations, Issues and Workarounds
-----------------------------------------

System Limitations
^^^^^^^^^^^^^^^^^^

None.

Known Issues
^^^^^^^^^^^^

None.

Workarounds
^^^^^^^^^^^

N/A.

Test Result
-----------

Please refer to `Testing Methodology <../installation/index.html#testing-methodology>`_.

References
----------

For more information on the OPNFV Euphrates release, please see:

http://www.opnfv.org/software

