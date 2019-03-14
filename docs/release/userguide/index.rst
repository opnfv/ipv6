.. _ipv6-userguide:

.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

====================================
Using IPv6 Feature of Hunter Release
====================================

:Abstract:

This section provides the users with:

* Gap Analysis regarding IPv6 feature requirements with OpenStack Rocky
  Official Release
* Gap Analysis regarding IPv6 feature requirements with Open Daylight Fluorine
  Official Release
* IPv6 Setup in Container Networking
* Use of Neighbor Discovery (ND) Proxy to connect IPv6-only container to
  external network
* Docker IPv6 Simple Cluster Topology
* Study and recommendation regarding Docker IPv6 NAT

The gap analysis serves as feature specific user guides and references when
as a user you may leverage the IPv6 feature in the platform and need to perform
some IPv6 related operations.

The IPv6 Setup in Container Networking serves as feature specific user guides
and references when as a user you may want to explore IPv6 in Docker container
environment. The use of NDP Proxying is explored to connect IPv6-only
containers to external network. The Docker IPv6 simple cluster topology is
studied with two Hosts, each with 2 Docker containers. Docker IPv6 NAT topic
is also explored.

For more information, please find `Neutron's IPv6 document for Rocky Release
<http://docs.openstack.org/neutron/rocky/admin/config-ipv6.html>`_.

.. toctree::
   :numbered:
   :maxdepth: 4

   ./gap-os-rocky.rst
   ./gap-odl-fluorine.rst
   ./ipv6-in-container-networking.rst
   ./icmpv6-and-ndp-proxying-for-docker-containers.rst
   ./docker-ipv6-simple-cluster-topology.rst
   ./docker-ipv6-nat.rst
