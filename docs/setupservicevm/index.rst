==========================================
Setting Up a Service VM as an IPv6 vRouter
==========================================

:Project: IPv6, http://wiki.opnfv.org/ipv6_opnfv_project
:Editors: Bin Hu (AT&T)
:Authors: Bin Hu (AT&T), Sridhar Gaddam (RedHat)

:Abstract: This document provides the users with installation guidelines to create a Service VM as
           an IPv6 vRouter in OPNFV environment, i.e. integrated OpenStack with Open Daylight
           environment. There are three scenarios. Scenario 1 is pre-OPNFV environment, i.e. a native
           OpenStack environment without Open Daylight Controller. Scenario 2 is an OPNFV environment
           where OpenStack is integrated with Open Daylight Official Lithium Release which not only
           does not support IPv6 L3 Routing but also has a bug in net-virt provider implementation
           that throws Java exception. Scenario 3 is similar to Scenario 2. However, Open Daylight
           Lithium is patched with a fix of Java exception. The complete set of instructions walk
           you through every step of preparing the infrastructure, setting up Open Daylight and
           OpenStack, creating service VM and IPv6 subnet, and testing and validating the setup.

.. toctree::
   :numbered:
   :maxdepth: 4

   5-ipv6-configguide-scenario-1-native-os.rst
   scenario-2.rst
