=====================================
IPv6 Gap Analysis with OpenStack Kilo
=====================================

This section provides users with IPv6 gap analysis regarding feature requirement with
OpenStack Neutron in Kilo Official Release. The following table lists the use cases / feature
requirements of VIM-agnostic IPv6 functionality, including infrastructure layer and VNF
(VM) layer, and its gap analysis with OpenStack Neutron in Kilo Official Release.

+-----------------------------------------------------------+-------------------------+--------------------------------------------------------------------+
|Use Case / Requirement                                     |Supported in Kilo Neutron|Notes                                                               |
+===========================================================+=========================+====================================================================+
|All topologies work in a multi-tenant environment          |Yes                      |The IPv6 design is following the Neutron tenant networks model;     |
|                                                           |                         |dnsmasq is being used inside DHCP network namespaces, while radvd   |
|                                                           |                         |is being used inside Neutron routers namespaces to provide full     |
|                                                           |                         |isolation between tenants. Tenant isolation can be based on VLANs,  |
|                                                           |                         |GRE, or VXLAN encapsulation. In case of overlays, the transport     |
|                                                           |                         |network (and VTEPs) must be IPv4 based as of today.                 |
+-----------------------------------------------------------+-------------------------+--------------------------------------------------------------------+
|IPv6 VM to VM only                                         |Yes                      |It is possible to assign IPv6-only addresses to VMs. Both switching |
|                                                           |                         |(within VMs on the same tenant network) as well as east/west routing|
|                                                           |                         |(between different networks of the same tenant) are supported.      |
+-----------------------------------------------------------+-------------------------+--------------------------------------------------------------------+
|IPv6 external L2 VLAN directly attached to a VM            |Yes                      |IPv6 provider network model; RA messages from upstream (external)   |
|                                                           |                         |router are forwarded into the VMs                                   |
+-----------------------------------------------------------+-------------------------+--------------------------------------------------------------------+
|IPv6 subnet routed via L3 agent to an external IPv6 network|                         |Configuration is enhanced in Kilo to allow easier setup of the      |
|                                                           |1. Yes                   |upstream gateway, without the user forced to create an IPv6 subnet  |
|1. Both VLAN and overlay (e.g. GRE, VXLAN) subnet attached |                         |for the external network.                                           |
|   to VMs;                                                 |                         |                                                                    |
|2. Must be able to support multiple L3 agents for a given  |2. Yes                   |                                                                    |
|   external network to support scaling (neutron scheduler  |                         |                                                                    |
|   to assign vRouters to the L3 agents)                    |                         |                                                                    |
+-----------------------------------------------------------+-------------------------+--------------------------------------------------------------------+
|Ability for a NIC to support both IPv4 and IPv6 (dual      |                         |Dual-stack is supported in Neutron with the addition of             |
|stack) address.                                            |                         |`Multiple IPv6 Prefixes`_ Blueprint.                                |
|                                                           |                         |                                                                    |
|1. VM with a single interface associated with a network,   |1. Yes                   |.. _`Multiple IPv6 Prefixes`:                                       |
|   which is then associated with two subnets.              |                         |http://blueprints.launchpad.net/neutron/+spec/multiple-ipv6-prefixes|
|2. VM with two different interfaces associated with two    |2. Yes                   |                                                                    |
|   different networks and two different subnets.           |                         |                                                                    |
+-----------------------------------------------------------+-------------------------+--------------------------------------------------------------------+

