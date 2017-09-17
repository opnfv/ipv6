.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

===========================================
IPv6 Gap Analysis with Open Daylight Carbon
===========================================

This section provides users with IPv6 gap analysis regarding feature requirement with
Open Daylight Carbon Official Release. The following table lists the use cases / feature
requirements of VIM-agnostic IPv6 functionality, including infrastructure layer and VNF
(VM) layer, and its gap analysis with Open Daylight Carbon Official Release.

**Open Daylight Carbon Status**

There are 2 options in Open Daylight Carbon to provide Virtualized Networks:

1 ``Old Netvirt``: netvirt implementation used in Open Daylight Beryllium and Boron Release
  identified by feature ``odl-ovsdb-openstack``

2 ``New Netvirt``: netvirt implementation which will replace the Old Netvirt in the
  future releases based on a more modular design. It is identified by feature
  ``odl-netvirt-openstack``

Please **NOTE** that in terms of IPv6 support in OpenDaylight, there is no difference
between **Carbon** release and **Boron** release.

.. table::
  :class: longtable

  +--------------------------------------------------+-----------------------------------------------+--------------------------------------------------------------+
  |Use Case / Requirement                            |            Supported in ODL Carbon            |Notes                                                         |
  |                                                  +----------------------+------------------------+                                                              |
  |                                                  |     Old Netvirt      |      New Netvirt       |                                                              |
  |                                                  |(odl-ovsdb-openstack) |(odl-netvirt-openstack) |                                                              |
  +==================================================+======================+========================+==============================================================+
  |REST API support for IPv6 subnet creation in ODL  |Yes                   |Yes                     |Yes, it is possible to create IPv6 subnets in ODL using       |
  |                                                  |                      |                        |Neutron REST API.                                             |
  |                                                  |                      |                        |                                                              |
  |                                                  |                      |                        |For a network which has both IPv4 and IPv6 subnets, ODL       |
  |                                                  |                      |                        |mechanism driver will send the port information which         |
  |                                                  |                      |                        |includes IPv4/v6 addresses to ODL Neutron northbound API.     |
  |                                                  |                      |                        |When port information is queried, it displays IPv4 and IPv6   |
  |                                                  |                      |                        |addresses.                                                    |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |IPv6 Router support in ODL:                       |**No**                |Yes                     |                                                              |
  |                                                  |                      |                        |                                                              |
  |1. Communication between VMs on same network      |                      |                        |                                                              |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |IPv6 Router support in ODL:                       |**No**                |Yes                     |                                                              |
  |                                                  |                      |                        |                                                              |
  |2. Communication between VMs on different         |                      |                        |                                                              |
  |   networks connected to the same router          |                      |                        |                                                              |
  |   (east-west)                                    |                      |                        |                                                              |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |IPv6 Router support in ODL:                       |**No**                |**Work in Progress**    |Work in progress.                                             |
  |                                                  |                      |                        |                                                              |
  |3. External routing (north-south)                 |                      |                        |                                                              |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |IPAM: Support for IPv6 Address assignment modes.  |**No**                |Yes                     |ODL IPv6 Router supports all the IPv6 Address assignment      |
  |                                                  |                      |                        |modes along with Neutron DHCP Agent.                          |
  |1. SLAAC                                          |                      |                        |                                                              |
  |2. DHCPv6 Stateless                               |                      |                        |                                                              |
  |3. DHCPv6 Stateful                                |                      |                        |                                                              |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |When using ODL for L2 forwarding/tunneling, it is |Yes                   |Yes                     |                                                              |
  |compatible with IPv6.                             |                      |                        |                                                              |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |Full support for IPv6 matching (i.e. IPv6, ICMPv6,|**Partial**           |Yes                     |Security Groups for IPv6 is supported in the new NetVirt.     |
  |TCP, UDP) in security groups. Ability to control  |                      |                        |                                                              |
  |and manage all IPv6 security group capabilities   |                      |                        |                                                              |
  |via Neutron/Nova API (REST and CLI) as well as    |                      |                        |                                                              |
  |via Horizon                                       |                      |                        |                                                              |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |Shared Networks support                           |Yes                   |Yes                     |                                                              |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |IPv6 external L2 VLAN directly attached to a VM.  |**ToDo**              |**ToDo**                |                                                              |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |ODL on an IPv6 only Infrastructure.               |**No**                |**Work in Progress**    |Deploying OpenStack with ODL on an IPv6 only infrastructure   |
  |                                                  |                      |                        |where the API endpoints are all IPv6 addresses.               |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+
  |VxLAN Tunnels with IPv6 Endpoints                 |**No**                |**Work in Progress**    |The necessary patches are under review to support this use    |
  |                                                  |                      |                        |case for OVS 2.6 or higher version.                           |
  +--------------------------------------------------+----------------------+------------------------+--------------------------------------------------------------+

