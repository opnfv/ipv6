.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

=========================================
Install OPNFV on IPv6-Only Infrastructure
=========================================

This section provides instructions to install OPNFV on IPv6-only
Infrastructure. All underlay networks and API endpoints will be IPv6-only
except:

1. "admin" network in underlay/undercloud still has to be IPv4.
* It was due to lack of support of IPMI over IPv6 or PXE over IPv6.
* iPXE does support IPv6 now. Ironic has added support for booting
  nodes with IPv6.
* We are starting to work on enabling IPv6-only environment for all
  networks. For TripleO, this work is still ongoing.
2. Metadata server is still IPv4 only.

Except the limitations above, the use case scenario of the IPv6-only
infrastructure includes:

1. Support OPNFV deployment on an IPv6 only infrastructure.
2. Horizon/ODL-DLUX access using IPv6 address from an external host.
3. OpenStack API access using IPv6 addresses from various python-clients.
4. Ability to create Neutron Routers, IPv6 subnets (e.g. SLAAC/DHCPv6-Stateful/
   DHCPv6-Stateless) to support North-South traffic.
5. Inter VM communication (East-West routing) when VMs are spread
   across two compute nodes.
6. VNC access into a VM using IPv6 addresses.
7. IPv6 support in OVS VxLAN (and/or GRE) tunnel endpoints with OVS 2.6+.
8. IPv6 support in iPXE, and booting nodes with IPv6 (**NEW**).

-------------------------------------------
Install OPNFV in OpenStack-Only Environment
-------------------------------------------

**Apex Installer**:

.. code-block:: bash

    # HA, Virtual deployment in OpenStack-only environment
    ./opnfv-deploy -v -d /etc/opnfv-apex/os-nosdn-nofeature-ha.yaml \
    -n /etc/opnfv-apex/network_settings_v6.yaml

    # HA, Bare Metal deployment in OpenStack-only environment
    ./opnfv-deploy -d /etc/opnfv-apex/os-nosdn-nofeature-ha.yaml \
    -i <inventory file> -n /etc/opnfv-apex/network_settings_v6.yaml

    # Non-HA, Virtual deployment in OpenStack-only environment
    ./opnfv-deploy -v -d /etc/opnfv-apex/os-nosdn-nofeature-noha.yaml \
    -n /etc/opnfv-apex/network_settings_v6.yaml

    # Non-HA, Bare Metal deployment in OpenStack-only environment
    ./opnfv-deploy -d /etc/opnfv-apex/os-nosdn-nofeature-noha.yaml \
    -i <inventory file> -n /etc/opnfv-apex/network_settings_v6.yaml

    # Note:
    #
    # 1. Parameter ""-v" is mandatory for Virtual deployment
    # 2. Parameter "-i <inventory file>" is mandatory for Bare Metal deployment
    # 2.1 Refer to https://git.opnfv.org/cgit/apex/tree/config/inventory for examples of inventory file
    # 3. You can use "-n /etc/opnfv-apex/network_settings.yaml" for deployment in IPv4 infrastructure

Please **NOTE** that:

* You need to refer to **installer's documentation** for other necessary
  parameters applicable to your deployment.
* You need to refer to **Release Notes** and **installer's documentation** if
  there is any issue in installation.

--------------------------------------------------
Install OPNFV in OpenStack with ODL-L3 Environment
--------------------------------------------------

**Apex Installer**:

.. code-block:: bash

    # HA, Virtual deployment in OpenStack with Open Daylight L3 environment
    ./opnfv-deploy -v -d /etc/opnfv-apex/os-odl-nofeature-ha.yaml \
    -n /etc/opnfv-apex/network_settings_v6.yaml

    # HA, Bare Metal deployment in OpenStack with Open Daylight L3 environment
    ./opnfv-deploy -d /etc/opnfv-apex/os-odl-nofeature-ha.yaml \
    -i <inventory file> -n /etc/opnfv-apex/network_settings_v6.yaml

    # Non-HA, Virtual deployment in OpenStack with Open Daylight L3 environment
    ./opnfv-deploy -v -d /etc/opnfv-apex/os-odl-nofeature-noha.yaml \
    -n /etc/opnfv-apex/network_settings_v6.yaml

    # Non-HA, Bare Metal deployment in OpenStack with Open Daylight L3 environment
    ./opnfv-deploy -d /etc/opnfv-apex/os-odl-nofeature-noha.yaml \
    -i <inventory file> -n /etc/opnfv-apex/network_settings_v6.yaml

    # Note:
    #
    # 1. Parameter ""-v" is mandatory for Virtual deployment
    # 2. Parameter "-i <inventory file>" is mandatory for Bare Metal deployment
    # 2.1 Refer to https://git.opnfv.org/cgit/apex/tree/config/inventory for examples of inventory file
    # 3. You can use "-n /etc/opnfv-apex/network_settings.yaml" for deployment in IPv4 infrastructure

Please **NOTE** that:

* You need to refer to **installer's documentation** for other necessary
  parameters applicable to your deployment.
* You need to refer to **Release Notes** and **installer's documentation** if
  there is any issue in installation.

-------------------
Testing Methodology
-------------------

There are 2 levels of testing to validate the deployment.

++++++++++++++++++++++++++++++++++++++++++++
Underlay Testing for OpenStack API Endpoints
++++++++++++++++++++++++++++++++++++++++++++

**Underlay** Testing is to validate that API endpoints are listening on IPv6
addresses. Currently, we are only considering the **Underlay Testing** for
OpenStack API endpoints. The **Underlay Testing** for Open Daylight API
endpoints is for future release.

The **Underlay Testing** for OpenStack API endpoints can be as simple as
validating Keystone service, and as complete as validating each API endpoint.
It is important to reuse Tempest API testing. Currently:

* Apex Installer will change ``OS_AUTH_URL`` in ``overcloudrc`` during
  installation process. For example:
  ``export OS_AUTH_URL=http://[2001:db8::15]:5000/v2.0``.
  ``OS_AUTH_URL`` points to Keystone and Keystone catalog.
* When FuncTest runs Tempest for the first time, the ``OS_AUTH_URL`` is taken
  from the environment and placed automatically in ``Tempest.conf``.
* Under this circumstance, ``openstack catalog list`` will return IPv6 URL
  endpoints for all the services in catalog, including Nova, Neutron, etc,
  and covering public URLs, private URLs and admin URLs.
* Thus, as long as the IPv6 URL is given in the ``overclourc``, all the tests
  will use that (including Tempest).

Therefore Tempest API testing is reused to validate API endpoints are listening
on IPv6 addresses as stated above. They are part of OpenStack default Smoke
Tests, run in FuncTest and integrated into OPNFV's CI/CD environment.

+++++++++++++++
Overlay Testing
+++++++++++++++

**Overlay** Testing is to validate that IPv6 is supported in tenant networks,
subnets and routers. Both Tempest API testing and Tempest Scenario testing are
used in our Overlay Testing.

Tempest API testing validates that the Neutron API supports the creation of
IPv6 networks, subnets, routers, etc:

.. code-block:: bash

    tempest.api.network.test_networks.BulkNetworkOpsIpV6Test.test_bulk_create_delete_network
    tempest.api.network.test_networks.BulkNetworkOpsIpV6Test.test_bulk_create_delete_port
    tempest.api.network.test_networks.BulkNetworkOpsIpV6Test.test_bulk_create_delete_subnet
    tempest.api.network.test_networks.NetworksIpV6Test.test_create_update_delete_network_subnet
    tempest.api.network.test_networks.NetworksIpV6Test.test_external_network_visibility
    tempest.api.network.test_networks.NetworksIpV6Test.test_list_networks
    tempest.api.network.test_networks.NetworksIpV6Test.test_list_subnets
    tempest.api.network.test_networks.NetworksIpV6Test.test_show_network
    tempest.api.network.test_networks.NetworksIpV6Test.test_show_subnet
    tempest.api.network.test_networks.NetworksIpV6TestAttrs.test_create_update_delete_network_subnet
    tempest.api.network.test_networks.NetworksIpV6TestAttrs.test_external_network_visibility
    tempest.api.network.test_networks.NetworksIpV6TestAttrs.test_list_networks
    tempest.api.network.test_networks.NetworksIpV6TestAttrs.test_list_subnets
    tempest.api.network.test_networks.NetworksIpV6TestAttrs.test_show_network
    tempest.api.network.test_networks.NetworksIpV6TestAttrs.test_show_subnet
    tempest.api.network.test_ports.PortsIpV6TestJSON.test_create_port_in_allowed_allocation_pools
    tempest.api.network.test_ports.PortsIpV6TestJSON.test_create_port_with_no_securitygroups
    tempest.api.network.test_ports.PortsIpV6TestJSON.test_create_update_delete_port
    tempest.api.network.test_ports.PortsIpV6TestJSON.test_list_ports
    tempest.api.network.test_ports.PortsIpV6TestJSON.test_show_port
    tempest.api.network.test_routers.RoutersIpV6Test.test_add_multiple_router_interfaces
    tempest.api.network.test_routers.RoutersIpV6Test.test_add_remove_router_interface_with_port_id
    tempest.api.network.test_routers.RoutersIpV6Test.test_add_remove_router_interface_with_subnet_id
    tempest.api.network.test_routers.RoutersIpV6Test.test_create_show_list_update_delete_router
    tempest.api.network.test_security_groups.SecGroupIPv6Test.test_create_list_update_show_delete_security_group
    tempest.api.network.test_security_groups.SecGroupIPv6Test.test_create_show_delete_security_group_rule
    tempest.api.network.test_security_groups.SecGroupIPv6Test.test_list_security_groups

Tempest Scenario testing validates some specific overlay IPv6 scenarios
(i.e. use cases) as follows:

.. code-block:: bash

    tempest.scenario.test_network_v6.TestGettingAddress.test_dhcp6_stateless_from_os
    tempest.scenario.test_network_v6.TestGettingAddress.test_dualnet_dhcp6_stateless_from_os
    tempest.scenario.test_network_v6.TestGettingAddress.test_dualnet_multi_prefix_dhcpv6_stateless
    tempest.scenario.test_network_v6.TestGettingAddress.test_dualnet_multi_prefix_slaac
    tempest.scenario.test_network_v6.TestGettingAddress.test_dualnet_slaac_from_os
    tempest.scenario.test_network_v6.TestGettingAddress.test_multi_prefix_dhcpv6_stateless
    tempest.scenario.test_network_v6.TestGettingAddress.test_multi_prefix_slaac
    tempest.scenario.test_network_v6.TestGettingAddress.test_slaac_from_os

The above Tempest API testing and Scenario testing are quite comprehensive to
validate overlay IPv6 tenant networks. They are part of OpenStack default
Smoke Tests, run in FuncTest and integrated into OPNFV's CI/CD environment.

