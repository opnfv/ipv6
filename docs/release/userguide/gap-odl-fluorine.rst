.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

=============================================
IPv6 Gap Analysis with Open Daylight Fluorine
=============================================

This section provides users with IPv6 gap analysis regarding feature requirement with
Open Daylight Fluorine Official Release. The following table lists the use cases / feature
requirements of VIM-agnostic IPv6 functionality, including infrastructure layer and VNF
(VM) layer, and its gap analysis with Open Daylight Fluorine Official Release.

**Open Daylight Fluorine Status**

In Open Daylight Fluorine official release, the legacy ``Old Netvirt`` identified by feature
``odl-ovsdb-openstack`` is deprecated and no longer supported. The ``New Netvirt``
identified by feature ``odl-netvirt-openstack`` is used.

Two new features are supported in Open Daylight Fluorine official release:

* Support for advertising MTU info in IPv6 RAs
* IPv6 external connectivity for FLAT/VLAN based provider networks

.. table::
  :class: longtable

  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |Use Case / Requirement                            |Supported in ODL Fluorine|Notes                                                         |
  +==================================================+=========================+==============================================================+
  |REST API support for IPv6 subnet creation in ODL  |Yes                      |Yes, it is possible to create IPv6 subnets in ODL using       |
  |                                                  |                         |Neutron REST API.                                             |
  |                                                  |                         |                                                              |
  |                                                  |                         |For a network which has both IPv4 and IPv6 subnets, ODL       |
  |                                                  |                         |mechanism driver will send the port information which         |
  |                                                  |                         |includes IPv4/v6 addresses to ODL Neutron northbound API.     |
  |                                                  |                         |When port information is queried, it displays IPv4 and IPv6   |
  |                                                  |                         |addresses.                                                    |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |IPv6 Router support in ODL:                       |Yes                      |                                                              |
  |                                                  |                         |                                                              |
  |1. Communication between VMs on same network      |                         |                                                              |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |IPv6 Router support in ODL:                       |Yes                      |                                                              |
  |                                                  |                         |                                                              |
  |2. Communication between VMs on different         |                         |                                                              |
  |   networks connected to the same router          |                         |                                                              |
  |   (east-west)                                    |                         |                                                              |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |IPv6 Router support in ODL:                       |**NO**                   |This feature is targeted for Flourine Release.                |
  |                                                  |                         |In ODL Fluorine Release, RFE "IPv6 Inter-DC L3 North-South    |
  |3. External routing (north-south)                 |                         |Connectivity Using L3VPN Provider Network Types" Spec [1]_ is |
  |                                                  |                         |merged. But the code patch has not been merged yet.           |
  |                                                  |                         |On the other hand, "IPv6 Cluster Support" is available in     |
  |                                                  |                         |Fluorine Release [2]_. Basically, existing IPv6 features were |
  |                                                  |                         |enhanced to work in a three node ODL Clustered Setup.         |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |IPAM: Support for IPv6 Address assignment modes.  |Yes                      |ODL IPv6 Router supports all the IPv6 Address assignment      |
  |                                                  |                         |modes along with Neutron DHCP Agent.                          |
  |1. SLAAC                                          |                         |                                                              |
  |2. DHCPv6 Stateless                               |                         |                                                              |
  |3. DHCPv6 Stateful                                |                         |                                                              |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |When using ODL for L2 forwarding/tunneling, it is |Yes                      |                                                              |
  |compatible with IPv6.                             |                         |                                                              |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |Full support for IPv6 matching (i.e. IPv6, ICMPv6,|Yes                      |                                                              |
  |TCP, UDP) in security groups. Ability to control  |                         |                                                              |
  |and manage all IPv6 security group capabilities   |                         |                                                              |
  |via Neutron/Nova API (REST and CLI) as well as    |                         |                                                              |
  |via Horizon                                       |                         |                                                              |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |Shared Networks support                           |Yes                      |                                                              |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |IPv6 external L2 VLAN directly attached to a VM.  |Yes                      |Targeted for Flourine Release                                 |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |ODL on an IPv6 only Infrastructure.               |Yes                      |Deploying OpenStack with ODL on an IPv6 only infrastructure   |
  |                                                  |                         |where the API endpoints are all IPv6 addresses.               |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |VxLAN Tunnels with IPv6 Endpoints                 |Yes                      |                                                              |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |IPv6 L3VPN Dual Stack with Single router          |Yes                      |Refer to "Dual Stack VM support in OpenDaylight" Spec [3]_.   |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |IPv6 Inter Data Center using L3VPNs               |Yes                      |Refer to "IPv6 Inter-DC L3 North-South connectivity using     |
  |                                                  |                         |L3VPN provider network types" Spec [1]_.                      |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |Support for advertising MTU info in IPv6 RAs      |Yes                      |                                                              |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+
  |IPv6 external connectivity for FLAT/VLAN based    |Yes                      |                                                              |
  |provider networks                                 |                         |                                                              |
  +--------------------------------------------------+-------------------------+--------------------------------------------------------------+

.. [1] https://docs.opendaylight.org/projects/netvirt/en/stable-fluorine/specs/oxygen/ipv6-interdc-l3vpn.html
.. [2] http://git.opendaylight.org/gerrit/#/c/66707/
.. [3] https://docs.opendaylight.org/projects/netvirt/en/stable-fluorine/specs/oxygen/l3vpn-dual-stack-vms.html
