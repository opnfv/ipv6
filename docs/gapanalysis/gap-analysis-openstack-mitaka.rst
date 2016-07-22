.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

=======================================
IPv6 Gap Analysis with OpenStack Mitaka
=======================================

This section provides users with IPv6 gap analysis regarding feature requirement with
OpenStack Neutron in Mitaka Official Release. The following table lists the use cases / feature
requirements of VIM-agnostic IPv6 functionality, including infrastructure layer and VNF
(VM) layer, and its gap analysis with OpenStack Neutron in Mitaka Official Release.

.. table::
  :class: longtable

  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Use Case / Requirement                                     |Supported in Mitaka|Notes                                                               |
  +===========================================================+===================+====================================================================+
  |All topologies work in a multi-tenant environment          |Yes                |The IPv6 design is following the Neutron tenant networks model;     |
  |                                                           |                   |dnsmasq is being used inside DHCP network namespaces, while radvd   |
  |                                                           |                   |is being used inside Neutron routers namespaces to provide full     |
  |                                                           |                   |isolation between tenants. Tenant isolation can be based on VLANs,  |
  |                                                           |                   |GRE, or VXLAN encapsulation. In case of overlays, the transport     |
  |                                                           |                   |network (and VTEPs) must be IPv4 based as of today.                 |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |IPv6 VM to VM only                                         |Yes                |It is possible to assign IPv6-only addresses to VMs. Both switching |
  |                                                           |                   |(within VMs on the same tenant network) as well as east/west routing|
  |                                                           |                   |(between different networks of the same tenant) are supported.      |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |IPv6 external L2 VLAN directly attached to a VM            |Yes                |IPv6 provider network model; RA messages from upstream (external)   |
  |                                                           |                   |router are forwarded into the VMs                                   |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |IPv6 subnet routed via L3 agent to an external IPv6 network|                   |Configuration is enhanced since Kilo to allow easier setup of the   |
  |                                                           |1. Yes             |upstream gateway, without the user being forced to create an IPv6   |
  |1. Both VLAN and overlay (e.g. GRE, VXLAN) subnet attached |                   |subnet for the external network.                                    |
  |   to VMs;                                                 |                   |                                                                    |
  |2. Must be able to support multiple L3 agents for a given  |2. Yes             |                                                                    |
  |   external network to support scaling (neutron scheduler  |                   |                                                                    |
  |   to assign vRouters to the L3 agents)                    |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Ability for a NIC to support both IPv4 and IPv6 (dual      |                   |Dual-stack is supported in Neutron with the addition of             |
  |stack) address.                                            |                   |``Multiple IPv6 Prefixes`` Blueprint                                |
  |                                                           |                   |                                                                    |
  |1. VM with a single interface associated with a network,   |1. Yes             |                                                                    |
  |   which is then associated with two subnets.              |                   |                                                                    |
  |2. VM with two different interfaces associated with two    |2. Yes             |                                                                    |
  |   different networks and two different subnets.           |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Support IPv6 Address assignment modes.                     |1. Yes             |                                                                    |
  |                                                           |                   |                                                                    |
  |1. SLAAC                                                   |2. Yes             |                                                                    |
  |2. DHCPv6 Stateless                                        |                   |                                                                    |
  |3. DHCPv6 Stateful                                         |3. Yes             |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Ability to create a port on an IPv6 DHCPv6 Stateful subnet |Yes                |                                                                    |
  |and assign a specific IPv6 address to the port and have it |                   |                                                                    |
  |taken out of the DHCP address pool.                        |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Ability to create a port with fixed_ip for a               |**No**             |The following patch disables this operation:                        |
  |SLAAC/DHCPv6-Stateless Subnet.                             |                   |https://review.openstack.org/#/c/129144/                            |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Support for private IPv6 to external IPv6 floating IP;     |**Rejected**       |Blueprint proposed in upstream and got rejected. General expectation|
  |Ability to specify floating IPs via Neutron API (REST and  |                   |is to avoid NAT with IPv6 by assigning GUA to tenant VMs. See       |
  |CLI) as well as via Horizon, including combination of      |                   |https://review.openstack.org/#/c/139731/ for discussion.            |
  |IPv6/IPv4 and IPv4/IPv6 floating IPs if implemented.       |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Provide IPv6/IPv4 feature parity in support for            |**To-Do**          |The L3 configuration should be transparent for the SR-IOV           |
  |pass-through capabilities (e.g., SR-IOV).                  |                   |implementation. SR-IOV networking support introduced in Juno based  |
  |                                                           |                   |on the ``sriovnicswitch`` ML2 driver is expected to work with IPv4  |
  |                                                           |                   |and IPv6 enabled VMs. We need to verify if it works or not.         |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Additional IPv6 extensions, for example: IPSEC, IPv6       |**No**             |It does not appear to be considered yet (lack of clear requirements)|
  |Anycast, Multicast                                         |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |VM access to the meta-data server to obtain user data, SSH |**No**             |This is currently not supported. Config-drive or dual-stack IPv4 /  |
  |keys, etc. using cloud-init with IPv6 only interfaces.     |                   |IPv6 can be used as a workaround (so that the IPv4 network is used  |
  |                                                           |                   |to obtain connectivity with the metadata service)                   |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Full support for IPv6 matching (i.e., IPv6, ICMPv6, TCP,   |Yes                |                                                                    |
  |UDP) in security groups. Ability to control and manage all |                   |                                                                    |
  |IPv6 security group capabilities via Neutron/Nova API (REST|                   |                                                                    |
  |and CLI) as well as via Horizon.                           |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |During network/subnet/router create, there should be an    |Yes                |Two new Subnet attributes were introduced to control IPv6 address   |
  |option to allow user to specify the type of address        |                   |assignment options:                                                 |
  |management they would like. This includes all options      |                   |                                                                    |
  |including those low priority if implemented (e.g., toggle  |                   |* ``ipv6-ra-mode``: to determine who sends Router Advertisements;   |
  |on/off router and address prefix advertisements); It must  |                   |                                                                    |
  |be supported via Neutron API (REST and CLI) as well as via |                   |* ``ipv6-address-mode``: to determine how VM obtains IPv6 address,  |
  |Horizon                                                    |                   |  default gateway, and/or optional information.                     |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Security groups anti-spoofing: Prevent VM from using a     |Yes                |                                                                    |
  |source IPv6/MAC address which is not assigned to the VM    |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Protect tenant and provider network from rogue RAs         |Yes                |When using a tenant network, Neutron is going to automatically      |
  |                                                           |                   |handle the filter rules to allow connectivity of RAs to the VMs only|
  |                                                           |                   |from the Neutron router port; with provider networks, users are     |
  |                                                           |                   |required to specify the LLA of the upstream router during the subnet|
  |                                                           |                   |creation, or otherwise manually edit the security-groups rules to   |
  |                                                           |                   |allow incoming traffic from this specific address.                  |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Support the ability to assign multiple IPv6 addresses to   |Yes                |                                                                    |
  |an interface; both for Neutron router interfaces and VM    |                   |                                                                    |
  |interfaces.                                                |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Ability for a VM to support a mix of multiple IPv4 and IPv6|Yes                |                                                                    |
  |networks, including multiples of the same type.            |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Support for IPv6 Prefix Delegation.                        |Yes                |Partial support in Mitaka                                           |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |Distributed Virtual Routing (DVR) support for IPv6         |**No**             |Blueprint proposed upstream, pending discussion.                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |IPv6 First-Hop Security, IPv6 ND spoofing                  |Yes                |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
  |IPv6 support in Neutron Layer3 High Availability           |Yes                |                                                                    |
  |(keepalived+VRRP).                                         |                   |                                                                    |
  +-----------------------------------------------------------+-------------------+--------------------------------------------------------------------+
