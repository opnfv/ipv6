=====================================
IPv6 Gap Analysis with OpenStack Kilo
=====================================

This section provides users with IPv6 gap analysis regarding feature requirement with
OpenStack Neutron in Kilo Official Release. The following table lists the use cases / feature
requirements of VIM-agnostic IPv6 functionality, including infrastructure layer and VNF
(VM) layer, and its gap analysis with OpenStack Neutron in Kilo Official Release.

.. table::
  :class: longtable

  +-------------------------------------+-------------------------+---------------------------------+
  |Use Case / Requirement               |Supported in Neutron     |Notes                            |
  +=====================================+=========================+=================================+
  |All topologies work in a multi-tenant|Yes                      |The IPv6 design is following the |
  |environment                          |                         |Neutron tenant networks model;   |
  |                                     |                         |dnsmasq is being used inside DHCP|
  |                                     |                         |network namespaces, while radvd  |
  |                                     |                         |is being used inside Neutron     |
  |                                     |                         |routers namespaces to provide    |
  |                                     |                         |full isolation between tenants.  |
  |                                     |                         |Tenant isolation can be based on |
  |                                     |                         |VLANs, GRE, or VXLAN             |
  |                                     |                         |encapsulation. In case of        |
  |                                     |                         |overlays, the transport network  |
  |                                     |                         |(and VTEPs) must be IPv4 based as|
  |                                     |                         |of today.                        |
  +-------------------------------------+-------------------------+---------------------------------+
  |IPv6 VM to VM only                   |Yes                      |It is possible to assign IPv6-   |
  |                                     |                         |only addresses to VMs. Both      |
  |                                     |                         |switching (within VMs on the same|
  |                                     |                         |tenant network) as well as east/ |
  |                                     |                         |west routing (between different  |
  |                                     |                         |networks of the same tenant) are |
  |                                     |                         |supported.                       |
  +-------------------------------------+-------------------------+---------------------------------+
  |IPv6 external L2 VLAN directly       |Yes                      |IPv6 provider network model; RA  |
  |attached to a VM                     |                         |messages from upstream (external)|
  |                                     |                         |router are forwarded into the VMs|
  +-------------------------------------+-------------------------+---------------------------------+
  |IPv6 subnet routed via L3 agent to an|                         |Configuration is enhanced in Kilo|
  |external IPv6 network                |                         |to allow easier setup of the     |
  |                                     |1. Yes                   |upstream gateway, without the    |
  |1. Both VLAN and overlay (e.g. GRE,  |                         |user forced to create an IPv6    |
  |   VXLAN) subnet attached to VMs;    |                         |subnet for the external network. |
  |2. Must be able to support multiple  |2. Yes                   |                                 |
  |   L3 agents for a given external    |                         |                                 |
  |   network to support scaling        |                         |                                 |
  |   (neutron scheduler to assign      |                         |                                 |
  |   vRouters to the L3 agents)        |                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |Ability for a NIC to support both    |                         |Dual-stack is supported in       |
  |IPv4 and IPv6 (dual stack) address.  |                         |Neutron with the addition of     |
  |                                     |                         |``Multiple IPv6 Prefixes``       |
  |1. VM with a single interface        |1. Yes                   |Blueprint                        |
  |   associated with a network, which  |                         |                                 |
  |   is then associated with two       |                         |                                 |
  |   subnets                           |                         |                                 |
  |2. VM with two different interfaces  |2. Yes                   |                                 |
  |   associated with two different     |                         |                                 |
  |   networks and two different subnets|                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |Support IPv6 Address assignment modes|1. Yes                   |                                 |
  |                                     |                         |                                 |
  |1. SLAAC                             |2. Yes                   |                                 |
  |2. DHCPv6 Stateless                  |                         |                                 |
  |3. DHCPv6 Stateful                   |3. Yes                   |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |Ability to create a port on an IPv6  |Yes                      |                                 |
  |DHCPv6 Stateful subnet and assign a  |                         |                                 |
  |specific IPv6 address to the port and|                         |                                 |
  |have it taken out of the DHCP address|                         |                                 |
  |pool.                                |                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |Ability to create a port with        |**No**                   |The following patch disables this|
  |fixed_ip for a SLAAC/DHCPv6-Stateless|                         |operation: https://review.opensta|
  |Subnet.                              |                         |ck.org/#/c/129144/               |
  +-------------------------------------+-------------------------+---------------------------------+
  |Support for private IPv6 to external |**Rejected**             |Blueprint proposed in upstream   |
  |IPv6 floating IP; Ability to specify |                         |and got rejected. General        |
  |floating IPs via Neutron API (REST   |                         |expectation is to avoid NAT with |
  |and CLI) as well as via Horizon,     |                         |IPv6 by assigning GUA to tenant  |
  |including combination of IPv6/IPv4   |                         |VMs. See https://review.openstack|
  |and IPv4/IPv6 floating IPs if        |                         |.org/#/c/139731/ for discussion. |
  |implemented.                         |                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |Provide IPv6/IPv4 feature parity in  |**To-Do**                |The L3 configuration should be   |
  |support for pass-through capabilities|                         |transparent for the SR-IOV       |
  |(e.g., SR-IOV).                      |                         |implementation. SR-IOV networking|
  |                                     |                         |support introduced in Juno based |
  |                                     |                         |on the ``sriovnicswitch`` ML2    |
  |                                     |                         |driver is expected to work with  |
  |                                     |                         |IPv4 and IPv6 enabled VMs. We    |
  |                                     |                         |need to verify if it works or not|
  +-------------------------------------+-------------------------+---------------------------------+
  |Additional IPv6 extensions, for      |**No**                   |It does not appear to be         |
  |example: IPSEC, IPv6 Anycast,        |                         |considered yet (lack of clear    |
  |Multicast                            |                         |requirements)                    |
  +-------------------------------------+-------------------------+---------------------------------+
  |VM access to the meta-data server to |**No**                   |This is currently not supported. |
  |obtain user data, SSH keys, etc.     |                         |Config-drive or dual-stack IPv4/ |
  |using cloud-init with IPv6 only      |                         |IPv6 can be used as a workaround |
  |interfaces.                          |                         |(so that the IPv4 network is used|
  |                                     |                         |to obtain connectivity with the  |
  |                                     |                         |metadata service)                |
  +-------------------------------------+-------------------------+---------------------------------+
  |Full support for IPv6 matching (i.e.,|Yes                      |                                 |
  |IPv6, ICMPv6, TCP, UDP) in security  |                         |                                 |
  |groups. Ability to control and manage|                         |                                 |
  |all IPv6 security group capabilities |                         |                                 |
  |via Neutron/Nova API (REST and CLI)  |                         |                                 |
  |as well as via Horizon.              |                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |During network/subnet/router create, |Yes                      |Two new Subnet attributes were   |
  |there should be an option to allow   |                         |introduced to control IPv6       |
  |user to specify the type of address  |                         |address assignment options:      |
  |management they would like. This     |                         |                                 |
  |includes all options including those |                         |* ``ipv6-ra-mode``: to determine |
  |low priority if implemented (e.g.,   |                         |  who sends Router Advertisements|
  |toggle on/off router and address     |                         |                                 |
  |prefix advertisements); It must be   |                         |* ``ipv6-address-mode``: to      |
  |supported via Neutron API (REST and  |                         |  determine how VM obtains IPv6  |
  |CLI) as well as via Horizon          |                         |  address, default gateway, and/ |
  |                                     |                         |  or optional information.       |
  +-------------------------------------+-------------------------+---------------------------------+
  |Security groups anti-spoofing:       |Yes                      |                                 |
  |Prevent VM from using a source       |                         |                                 |
  |IPv6/MAC address which is not        |                         |                                 |
  |assigned to the VM                   |                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |Protect tenant and provider network  |Yes                      |When using a tenant network,     |
  |from rough RAs                       |                         |Neutron is going to automatically|
  |                                     |                         |handle the filter rules to allow |
  |                                     |                         |connectivity of RAs to the VMs   |
  |                                     |                         |only from the Neutron router     |
  |                                     |                         |port; with provider networks,    |
  |                                     |                         |users are required to specify the|
  |                                     |                         |LLA of the upstream router during|
  |                                     |                         |the subnet creation, or otherwise|
  |                                     |                         |manually edit the security-groups|
  |                                     |                         |rules to allow incoming traffic  |
  |                                     |                         |from this specific address.      |
  +-------------------------------------+-------------------------+---------------------------------+
  |Support the ability to assign        |Yes                      |                                 |
  |multiple IPv6 addresses to an        |                         |                                 |
  |interface; both for Neutron router   |                         |                                 |
  |interfaces and VM interfaces.        |                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |Ability for a VM to support a mix of |Yes                      |                                 |
  |multiple IPv4 and IPv6 networks,     |                         |                                 |
  |including multiples of the same type.|                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |Support for IPv6 Prefix Delegation.  |**Roadmap**              |Some partial support is available|
  |                                     |                         |in Liberty release               |
  +-------------------------------------+-------------------------+---------------------------------+
  |Distributed Virtual Routing (DVR)    |**No**                   |Blueprint proposed upstream,     |
  |support for IPv6                     |                         |pending discussion               |
  +-------------------------------------+-------------------------+---------------------------------+
  |IPv6 First-Hop Security, IPv6 ND     |**Roadmap**              |Supported in Liberty release     |
  |spoofing.                            |                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
  |IPv6 support in Neutron Layer3 High  |Yes                      |                                 |
  |Availability (keepalived+VRRP).      |                         |                                 |
  +-------------------------------------+-------------------------+---------------------------------+
