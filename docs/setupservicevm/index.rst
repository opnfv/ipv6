.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

==========================================
Setting Up a Service VM as an IPv6 vRouter
==========================================

:Project: IPv6, http://wiki.opnfv.org/ipv6_opnfv_project
:Editors: Bin Hu (AT&T), Sridhar Gaddam (RedHat)
:Authors: Sridhar Gaddam (RedHat), Bin Hu (AT&T)

:Abstract:

This document provides the users with installation guidelines to create a Service VM as
an IPv6 vRouter in OPNFV environment, i.e. integrated OpenStack with Open Daylight
environment. There are three scenarios.

* Scenario 1 is pre-OPNFV environment, i.e. a native OpenStack environment
  without Open Daylight Controller.
* Scenario 2 is an OPNFV environment where OpenStack is integrated with
  Open Daylight Official Lithium Release. In this setup we use ODL for "Layer 2 connectivity"
  and Neutron L3 agent for "Layer 3 routing". Because of a bug, which got fixed recently
  and is not part of ODL SR3, we will have to manually execute certain commands to simulate
  an external IPv6 Router in this setup.
* Scenario 3 is similar to Scenario 2. However, we use an Open Daylight Lithium
  controller which is built from the latest stable/Lithium branch which includes the fix.
  In this scenario, we can fully automate the setup similar to Scenario 1.

Please **NOTE** that the instructions in this document assume the deployment model of single
controller node. In case of HA (High Availability) deployment model where multiple controller
nodes are used, the setup procedure is the same. In particular:

* There is **No Impact** on Scenario 1 and Scenario 3.
* For Scenario 2, when ``ipv6-router`` is created in step **SETUP-SVM-11**, it could be
  created in any of the controller node. Thus you need to identify in which controller node
  ``ipv6-router`` is created in order to manually spawn ``radvd`` daemon inside the
  ``ipv6-router`` namespace in steps **SETUP-SVM-24** through **SETUP-SVM-30**.


.. toctree::
   :numbered:
   :maxdepth: 4

   architecture-design.rst
   5-ipv6-configguide-scenario-1-native-os.rst
   scenario-2.rst
   scenario-3.rst
   topology-after-setup.rst
