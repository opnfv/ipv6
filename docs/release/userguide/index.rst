.. _ipv6-userguide:

.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

====================================
Using IPv6 Feature of Gambia Release
====================================

:Abstract:

This section provides the users with:

* Gap Analysis regarding IPv6 feature requirements with OpenStack Queens
  Official Release
* Gap Analysis regarding IPv6 feature requirements with Open Daylight Oxygen
  Official Release
* IPv6 Setup in Container Networking
* Use of Neighbor Discovery (ND) Proxy to connect IPv6-only container to
  external network

The gap analysis serves as feature specific user guides and references when
as a user you may leverage the IPv6 feature in the platform and need to perform
some IPv6 related operations.

The IPv6 Setup in Container Networking serves as feature specific user guides
and references when as a user you may want to explore IPv6 in Docker container
environment. The use of NDP Proxying is also explored to connect IPv6-only
containers to external network.

For more information, please find `Neutron's IPv6 document for Queens Release
<http://docs.openstack.org/neutron/queens/admin/config-ipv6.html>`_.

.. toctree::
   :numbered:
   :maxdepth: 4

   ./gap-os-queens.rst
   ./gap-odl-oxygen.rst
   ./ipv6-in-container-networking.rst
   ./icmpv6-and-ndp-proxying-for-docker-containers.rst
