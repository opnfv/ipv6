============================================
IPv6 Gap Analysis with Open Daylight Lithium
============================================

This section provides users with IPv6 gap analysis regarding feature requirement with
Open Daylight Lithium Official Release. The following table lists the use cases / feature
requirements of VIM-agnostic IPv6 functionality, including infrastructure layer and VNF
(VM) layer, and its gap analysis with Open Daylight Lithium Official Release.

+-------------------------------------------------------------+------------------------+--------------------------------------------------------------------------------+
|Use Case / Requirement                                       |Supported in ODL Lithium|Notes                                                                           |
+=============================================================+========================+================================================================================+
|REST API support for IPv6 subnet creation in ODL             |Yes                     |Yes, it is possible to create IPv6 subnets in ODL using Neutron REST API.       |
|                                                             |                        |                                                                                |
|                                                             |                        |For a network which has both IPv4 and IPv6 subnets, ODL mechanism driver        |
|                                                             |                        |will send the port information which includes IPv4/v6 addresses to ODL          |
|                                                             |                        |Neutron northbound API. When port information is queried it displays IPv4       |
|                                                             |                        |and IPv6 addresses. However, in Lithium release, ODL net-virt provider          |
|                                                             |                        |does not support IPv6 features (i.e., the actual functionality is missing       |
|                                                             |                        |and would be available only in the later releases of ODL).                      |
+-------------------------------------------------------------+------------------------+--------------------------------------------------------------------------------+
|IPv6 Router support in ODL                                   |**No**                  |ODL net-virt provider in Lithium release only supports IPv4 Router.             |
|                                                             |                        |Support for IPv6 Router is planned using ``Routing Manager`` as part of         |
|1. Communication between VMs on same compute node            |                        |Beryllium Release. In the meantime, if IPv6 Routing is necessary, we can        |
|2. Communication between VMs on different compute nodes      |                        |use ODL for L2 connectivity and Neutron L3 agent for IPv4/v6 routing.           |
|   (east-west)                                               |                        |                                                                                |
|3. External routing (north-south)                            |                        |**Note**: In Lithium SR3 release, we have the following `issue                  |
|                                                             |                        |<http://lists.opendaylight.org/pipermail/ovsdb-dev/2015-November/002288.html>`_,|
|                                                             |                        |which is fixed `upstream <http://git.opendaylight.org/gerrit/#/c/30253/4>`_     |
|                                                             |                        |and `back-ported <http://git.opendaylight.org/gerrit/#/c/30474>`_ to            |
|                                                             |                        |``stable/lithium`` branch on December 15th, 2015.                               |
+-------------------------------------------------------------+------------------------+--------------------------------------------------------------------------------+
|IPAM: Support for IPv6 Address assignment modes.             |**No**                  |Although it is possible to create different types of IPv6 subnets in ODL,       |
|                                                             |                        |ODL_L3 would have to implement the IPv6 Router that can send out Router         |
|1. SLAAC                                                     |                        |Advertisements based on the IPv6 addressing mode. Router Advertisement          |
|2. DHCPv6 Stateless                                          |                        |is also necessary for VMs to configure the default route.                       |
|3. DHCPv6 Stateful                                           |                        |                                                                                |
|                                                             |                        |This could be part of ``Routing Manager`` in ``Beryllium`` release.             |
+-------------------------------------------------------------+------------------------+--------------------------------------------------------------------------------+
|When using ODL for L2 forwarding/tunneling, is it compatible |Yes                     |                                                                                |
|with IPv6.                                                   |                        |                                                                                |
+-------------------------------------------------------------+------------------------+--------------------------------------------------------------------------------+
|Full support for IPv6 matching (i.e., IPv6, ICMPv6, TCP, UDP)|**No**                  |Security Groups for IPv6 are currently not supported.                           |
|in security groups. Ability to control and manage all IPv6   |                        |                                                                                |
|security group capabilities via Neutron/Nova API (REST and   |                        |                                                                                |
|CLI) as well as via Horizon.                                 |                        |                                                                                |
+-------------------------------------------------------------+------------------------+--------------------------------------------------------------------------------+
|Shared Networks support                                      |**No**                  |ODL currently assumes a single tenant to network mapping and does not           |
|                                                             |                        |support shared networks among tenants.                                          |
+-------------------------------------------------------------+------------------------+--------------------------------------------------------------------------------+
|IPv6 external L2 VLAN directly attached to a VM.             |**ToDo**                |                                                                                |
+-------------------------------------------------------------+------------------------+--------------------------------------------------------------------------------+
|ODL on an IPv6 only Infrastructure.                          |**ToDo**                |Deploying OpenStack with ODL on an IPv6 only infrastructure where the API       |
|                                                             |                        |endpoints are all IPv6 addresses.                                               |
+-------------------------------------------------------------+------------------------+--------------------------------------------------------------------------------+
