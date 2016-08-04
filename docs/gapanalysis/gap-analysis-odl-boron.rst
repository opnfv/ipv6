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

.. table::
  :class: longtable

  +-------------------------------------------------------------+----------------------+--------------------------------------------------------------------------+
  |Use Case / Requirement                                       |Supported in ODL Boron|Notes                                                                     |
  |                                                             +----------+-----------+                                                                          |
  |                                                             |   1      |     2     |                                                                          |
  +=============================================================+==========+===========+==========================================================================+
  |REST API support for IPv6 subnet creation in ODL             |Yes       |           |Yes, it is possible to create IPv6 subnets in ODL using Neutron REST API. |
  |                                                             |          |           |                                                                          |
  |                                                             |          |           |For a network which has both IPv4 and IPv6 subnets, ODL mechanism driver  |
  |                                                             |          |           |will send the port information which includes IPv4/v6 addresses to ODL    |
  |                                                             |          |           |Neutron northbound API. When port information is queried it displays IPv4 |
  |                                                             |          |           |and IPv6 addresses. However, in Boron release, ODL net-virt provider      |
  |                                                             |          |           |does not support IPv6 features (i.e., the actual functionality is missing |
  |                                                             |          |           |and would be available only in the later releases of ODL).                |
  +-------------------------------------------------------------+----------+-----------+--------------------------------------------------------------------------+
  |IPv6 Router support in ODL                                   |**No**                |ODL net-virt provider in Boron release only supports IPv4 Router.         |
  |                                                             |                      |                                                                          |
  |1. Communication between VMs on same compute node            |                      |In the meantime, if IPv6 Routing is necessary, we can use ODL for L2      |
  |2. Communication between VMs on different compute nodes      |                      |connectivity and Neutron L3 agent for IPv4/v6 routing.                    |
  |   (east-west)                                               |                      |                                                                          |
  |3. External routing (north-south)                            |                      |                                                                          |
  +-------------------------------------------------------------+----------------------+--------------------------------------------------------------------------+
  |IPAM: Support for IPv6 Address assignment modes.             |**No**                |Although it is possible to create different types of IPv6 subnets in ODL, |
  |                                                             |                      |ODL_L3 would have to implement the IPv6 Router that can send out Router   |
  |1. SLAAC                                                     |                      |Advertisements based on the IPv6 addressing mode. Router Advertisement    |
  |2. DHCPv6 Stateless                                          |                      |is also necessary for VMs to configure the default route.                 |
  |3. DHCPv6 Stateful                                           |                      |                                                                          |
  +-------------------------------------------------------------+----------------------+--------------------------------------------------------------------------+
  |When using ODL for L2 forwarding/tunneling, it is compatible |Yes                   |                                                                          |
  |with IPv6.                                                   |                      |                                                                          |
  +-------------------------------------------------------------+----------------------+--------------------------------------------------------------------------+
  |Full support for IPv6 matching (i.e., IPv6, ICMPv6, TCP, UDP)|**No**                |Security Groups for IPv6 is a work in progress.                           |
  |in security groups. Ability to control and manage all IPv6   |                      |                                                                          |
  |security group capabilities via Neutron/Nova API (REST and   |                      |                                                                          |
  |CLI) as well as via Horizon.                                 |                      |                                                                          |
  +-------------------------------------------------------------+----------------------+--------------------------------------------------------------------------+
  |Shared Networks support                                      |Yes                   |                                                                          |
  +-------------------------------------------------------------+----------------------+--------------------------------------------------------------------------+
  |IPv6 external L2 VLAN directly attached to a VM.             |**ToDo**              |                                                                          |
  +-------------------------------------------------------------+----------------------+--------------------------------------------------------------------------+
  |ODL on an IPv6 only Infrastructure.                          |**ToDo**              |Deploying OpenStack with ODL on an IPv6 only infrastructure where the API |
  |                                                             |                      |endpoints are all IPv6 addresses.                                         |
  +-------------------------------------------------------------+----------------------+--------------------------------------------------------------------------+
