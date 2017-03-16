.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

================================
OPNFV IPv6 Project Release Notes
================================

This document provides the release notes for Danube of IPv6 Project.

.. contents::
   :depth: 3
   :local:


Version History
---------------

+--------------------+--------------------+--------------------+--------------------+
| **Date**           | **Ver.**           | **Author**         | **Comment**        |
|                    |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| 2017-02-16         | 0.1.0              | Bin Hu             | First draft        |
|                    |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| 2017-03-15         | 0.2.0              | Bin Hu             | Baseline draft     |
|                    |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
|                    | 1.0                |                    |                    |
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

For details, please refer to our `**User Guide** <../release_userguide/index.html>`_.

Summary
-------

This is the Danube release of the IPv6 feature as part of OPNFV, including:

* Installation of OPNFV on IPv6-Only Infrastructure by Apex Installer
* Configuration of setting up a Service VM as an IPv6 vRouter
* User Guide, which analyzes the gap of IPv6 support in OpenStack Newton
  and OpenDaylight Boron.

Please refer to our:

* `**Install Guide** <release_installation/index.html>`_
* `**Configuration Guide** <release_configguide/index.html>`_
* `**User Guide** <../release_userguide/index.html>`_

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

Please refer to `**Testing Methodology** <../release_installation/index.html#testing-methodology>`_.

References
----------

For more information on the OPNFV Danube release, please see:

http://opnfv.org/danube

