.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

===============================================================
Scenario 2 - OpenStack + Open Daylight Lithium Official Release
===============================================================

Scenario 2 is the environment of OpenStack + Open Daylight Lithium SR3 Official Release.
Because Lithium SR3 Official Release does not support IPv6 L3 Routing, we need to enable
Neutron L3 Agent instead of Open Daylight L3 function, while we still use Open Daylight for
L2 switching. Because there is a bug in net-virt provider implementation, we need to use
manual configuration to simulate IPv6 external router in our setup.

Please note that although the instructions are based on OpenStack Kilo, they can be applied to Liberty in the same way.

.. toctree::
   :numbered:
   :maxdepth: 4

   0-ipv6-configguide-prep-infra.rst
   1-ipv6-configguide-odl-setup.rst
   2-ipv6-configguide-os-controller.rst
   3-ipv6-configguide-os-compute.rst
   4-ipv6-configguide-servicevm.rst
