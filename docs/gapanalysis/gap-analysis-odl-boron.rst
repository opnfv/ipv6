.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

==========================================
IPv6 Gap Analysis with Open Daylight Boron
==========================================

This section provides users with IPv6 gap analysis regarding feature requirement with
Open Daylight Boron Official Release. The following table lists the use cases / feature
requirements of VIM-agnostic IPv6 functionality, including infrastructure layer and VNF
(VM) layer, and its gap analysis with Open Daylight Boron Official Release.

**Open Daylight Boron Status**

There are 2 options in Open Daylight Boron to provide Virtualized Networks:

1 ''Old Netvirt'': netvirt implementation used in Open Daylight Beryllium Release
  identified by feature ''odl-ovsdb-openstack''

2 ''New Netvirt'': netvirt implementation which will replace the Old Netvirt in the future
  releases based on a more modular design. It is identified by feature ''odl-netvirt-openstack''

.. table::
  :class: longtable

  +-------------------------------------------------------------+---------------------------------------------+--------------------------------------------------------------------------+
  |Use Case / Requirement                                       |           Supported in ODL Boron            |Notes                                                                     |
  |                                                             +---------------------+-----------------------+                                                                          |
  |                                                             |     Old Netvirt     |      New Netvirt      |                                                                          |
  |                                                             |(odl-ovsdb-openstack)|(odl-netvirt-openstack)|                                                                          |
  +=============================================================+=====================+=======================+==========================================================================+
  |REST API support for IPv6 subnet creation in ODL             |Yes                  |Yes                    |Yes, it is possible to create IPv6 subnets in ODL using Neutron REST API. |
  |                                                             |                     |                       |                                                                          |
  |                                                             |                     |                       |For a network which has both IPv4 and IPv6 subnets, ODL mechanism driver  |
  |                                                             |                     |                       |will send the port information which includes IPv4/v6 addresses to ODL    |
  |                                                             |                     |                       |Neutron northbound API. When port information is queried it displays IPv4 |
  |                                                             |                     |                       |and IPv6 addresses.                                                       |
  +-------------------------------------------------------------+---------------------+-----------------------+--------------------------------------------------------------------------+
  |IPv6 Router support in ODL                                   |**No**               |**Partial**            |IPv6 Router support is work in progress in ODL.                           |
  |                                                             |                     |                       |                                                                          |
  |1. Communication between VMs on same compute node            |                     |                       |Currently communication between VMs on the same network is supported, and |
  |2. Communication between VMs on different compute nodes      |                     |                       |the support for the other modes is work in progress                       |
  |   (east-west)                                               |                     |                       |                                                                          |
  |3. External routing (north-south)                            |                     |                       |                                                                          |
  +-------------------------------------------------------------+---------------------+-----------------------+--------------------------------------------------------------------------+
  |IPAM: Support for IPv6 Address assignment modes.             |**No**               |Yes                    |ODL IPv6 Router supports all the IPv6 Address assignment modes along with |
  |                                                             |                     |                       |Neutron DHCP Agent.                                                       |
  |1. SLAAC                                                     |                     |                       |                                                                          |
  |2. DHCPv6 Stateless                                          |                     |                       |                                                                          |
  |3. DHCPv6 Stateful                                           |                     |                       |                                                                          |
  +-------------------------------------------------------------+---------------------+-----------------------+--------------------------------------------------------------------------+
  |When using ODL for L2 forwarding/tunneling, it is compatible |Yes                  |Yes                    |                                                                          |
  |with IPv6.                                                   |                     |                       |                                                                          |
  +-------------------------------------------------------------+---------------------+-----------------------+--------------------------------------------------------------------------+
  |Full support for IPv6 matching (i.e., IPv6, ICMPv6, TCP, UDP)|**Partial**          |**Partial**            |Security Groups for IPv6 is a work in progress.                           |
  |in security groups. Ability to control and manage all IPv6   |                     |                       |                                                                          |
  |security group capabilities via Neutron/Nova API (REST and   |                     |                       |                                                                          |
  |CLI) as well as via Horizon.                                 |                     |                       |                                                                          |
  +-------------------------------------------------------------+---------------------+-----------------------+--------------------------------------------------------------------------+
  |Shared Networks support                                      |Yes                  |Yes                    |                                                                          |
  +-------------------------------------------------------------+---------------------+-----------------------+--------------------------------------------------------------------------+
  |IPv6 external L2 VLAN directly attached to a VM.             |**ToDo**             |**ToDo**               |                                                                          |
  +-------------------------------------------------------------+---------------------+-----------------------+--------------------------------------------------------------------------+
  |ODL on an IPv6 only Infrastructure.                          |**No**               |**Work in Progress**   |Deploying OpenStack with ODL on an IPv6 only infrastructure where the API |
  |                                                             |                     |                       |endpoints are all IPv6 addresses.                                         |
  +-------------------------------------------------------------+---------------------+-----------------------+--------------------------------------------------------------------------+
