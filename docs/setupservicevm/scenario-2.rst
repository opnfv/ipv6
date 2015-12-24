===============================================================
Scenario 2 - OpenStack + Open Daylight Lithium Official Release
===============================================================

Scenario 2 is the environment of OpenStack + Open Daylight Lithium Official Release. Because Lithium Official
Release does not support IPv6 L3 Routing, we need to enable Neutron L3 Agent instead of Open Daylight L3
function, while we still use Open Daylight for L2 switching. Because there is a bug in net-virt provider
implementation, we need to use manual configuration to work around this bug / Java exception.

.. toctree::
   :numbered:
   :maxdepth: 4

   0-ipv6-configguide-prep-infra.rst
   1-ipv6-configguide-odl-setup.rst
   2-ipv6-configguide-os-controller.rst
   3-ipv6-configguide-os-compute.rst
   4-ipv6-configguide-servicevm.rst
