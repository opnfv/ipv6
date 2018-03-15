.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

===============================================================
IPv6 Configuration - Setting Up a Service VM as an IPv6 vRouter
===============================================================

This section provides instructions to set up a service VM as an IPv6 vRouter using OPNFV Fraser Release
installers. Because Open Daylight no longer supports L2-only option, and there is only limited support of
IPv6 in L3 option of Open Daylight, setup of service VM as an IPv6 vRouter is only available under
pure/native OpenStack environment. The deployment model may be HA or non-HA. The infrastructure may be
bare metal or virtual environment.

****************************
Pre-configuration Activities
****************************

The configuration will work only in OpenStack-only environment.

Depending on which installer will be used to deploy OPNFV, each environment may be deployed
on bare metal or virtualized infrastructure. Each deployment may be HA or non-HA.

Refer to the previous installer configuration chapters, installations guide and release notes.

******************************************
Setup Manual in OpenStack-Only Environment
******************************************

If you intend to set up a service VM as an IPv6 vRouter in OpenStack-only environment of
OPNFV Fraser Release, please **NOTE** that:

* Because the anti-spoofing rules of Security Group feature in OpenStack prevents
  a VM from forwarding packets, we need to disable Security Group feature in the
  OpenStack-only environment.
* The hostnames, IP addresses, and username are for exemplary purpose in instructions.
  Please change as needed to fit your environment.
* The instructions apply to both deployment model of single controller node and
  HA (High Availability) deployment model where multiple controller nodes are used.

-----------------------------
Install OPNFV and Preparation
-----------------------------

**OPNFV-NATIVE-INSTALL-1**: To install OpenStack-only environment of OPNFV Fraser Release:

**Apex Installer**:

.. code-block:: bash

    # HA, Virtual deployment in OpenStack-only environment
    ./opnfv-deploy -v -d /etc/opnfv-apex/os-nosdn-nofeature-ha.yaml \
    -n /etc/opnfv-apex/network_setting.yaml

    # HA, Bare Metal deployment in OpenStack-only environment
    ./opnfv-deploy -d /etc/opnfv-apex/os-nosdn-nofeature-ha.yaml \
    -i <inventory file> -n /etc/opnfv-apex/network_setting.yaml

    # Non-HA, Virtual deployment in OpenStack-only environment
    ./opnfv-deploy -v -d /etc/opnfv-apex/os-nosdn-nofeature-noha.yaml \
    -n /etc/opnfv-apex/network_setting.yaml

    # Non-HA, Bare Metal deployment in OpenStack-only environment
    ./opnfv-deploy -d /etc/opnfv-apex/os-nosdn-nofeature-noha.yaml \
    -i <inventory file> -n /etc/opnfv-apex/network_setting.yaml

    # Note:
    #
    # 1. Parameter ""-v" is mandatory for Virtual deployment
    # 2. Parameter "-i <inventory file>" is mandatory for Bare Metal deployment
    # 2.1 Refer to https://git.opnfv.org/cgit/apex/tree/config/inventory for examples of inventory file
    # 3. You can use "-n /etc/opnfv-apex/network_setting_v6.yaml" for deployment in IPv6-only infrastructure

**Compass** Installer:

.. code-block:: bash

    # HA deployment in OpenStack-only environment
    export ISO_URL=file://$BUILD_DIRECTORY/compass.iso
    export OS_VERSION=${{COMPASS_OS_VERSION}}
    export OPENSTACK_VERSION=${{COMPASS_OPENSTACK_VERSION}}
    export CONFDIR=$WORKSPACE/deploy/conf/vm_environment
    ./deploy.sh --dha $CONFDIR/os-nosdn-nofeature-ha.yml \
    --network $CONFDIR/$NODE_NAME/network.yml

    # Non-HA deployment in OpenStack-only environment
    # Non-HA deployment is currently not supported by Compass installer

**Fuel** Installer:

.. code-block:: bash

    # HA deployment in OpenStack-only environment
    # Scenario Name: os-nosdn-nofeature-ha
    # Scenario Configuration File: ha_heat_ceilometer_scenario.yaml
    # You can use either Scenario Name or Scenario Configuration File Name in "-s" parameter
    sudo ./deploy.sh -b <stack-config-uri> -l <lab-name> -p <pod-name> \
    -s os-nosdn-nofeature-ha -i <iso-uri>

    # Non-HA deployment in OpenStack-only environment
    # Scenario Name: os-nosdn-nofeature-noha
    # Scenario Configuration File: no-ha_heat_ceilometer_scenario.yaml
    # You can use either Scenario Name or Scenario Configuration File Name in "-s" parameter
    sudo ./deploy.sh -b <stack-config-uri> -l <lab-name> -p <pod-name> \
    -s os-nosdn-nofeature-noha -i <iso-uri>

    # Note:
    #
    # 1. Refer to http://git.opnfv.org/cgit/fuel/tree/deploy/scenario/scenario.yaml for scenarios
    # 2. Refer to http://git.opnfv.org/cgit/fuel/tree/ci/README for description of
    #    stack configuration directory structure
    # 3. <stack-config-uri> is the base URI of stack configuration directory structure
    # 3.1 Example: http://git.opnfv.org/cgit/fuel/tree/deploy/config
    # 4. <lab-name> and <pod-name> must match the directory structure in stack configuration
    # 4.1 Example of <lab-name>: -l devel-pipeline
    # 4.2 Example of <pod-name>: -p elx
    # 5. <iso-uri> could be local or remote ISO image of Fuel Installer
    # 5.1 Example: http://artifacts.opnfv.org/fuel/euphrates/opnfv-euphrates.1.0.iso
    #
    # Please refer to Fuel Installer's documentation for further information and any update

**Joid** Installer:

.. code-block:: bash

    # HA deployment in OpenStack-only environment
    ./deploy.sh -o mitaka -s nosdn -t ha -l default -f ipv6

    # Non-HA deployment in OpenStack-only environment
    ./deploy.sh -o mitaka -s nosdn -t nonha -l default -f ipv6

Please **NOTE** that:

* You need to refer to **installer's documentation** for other necessary
  parameters applicable to your deployment.
* You need to refer to **Release Notes** and **installer's documentation** if there is
  any issue in installation.

**OPNFV-NATIVE-INSTALL-2**: Clone the following GitHub repository to get the
configuration and metadata files

.. code-block:: bash

    git clone https://github.com/sridhargaddam/opnfv_os_ipv6_poc.git \
    /opt/stack/opnfv_os_ipv6_poc

----------------------------------------------
Disable Security Groups in OpenStack ML2 Setup
----------------------------------------------

Please **NOTE** that although Security Groups feature has been disabled automatically
through ``local.conf`` configuration file by some installers such as ``devstack``, it is very likely
that other installers such as ``Apex``, ``Compass``, ``Fuel`` or ``Joid`` will enable Security
Groups feature after installation.

**Please make sure that Security Groups are disabled in the setup**

In order to disable Security Groups globally, please make sure that the settings in
**OPNFV-NATIVE-SEC-1** and **OPNFV-NATIVE-SEC-2** are applied, if they
are not there by default.

**OPNFV-NATIVE-SEC-1**: Change the settings in
``/etc/neutron/plugins/ml2/ml2_conf.ini`` as follows, if they are not there by default

.. code-block:: bash

    # /etc/neutron/plugins/ml2/ml2_conf.ini
    [securitygroup]
    enable_security_group = True
    firewall_driver = neutron.agent.firewall.NoopFirewallDriver
    [ml2]
    extension_drivers = port_security
    [agent]
    prevent_arp_spoofing = False

**OPNFV-NATIVE-SEC-2**: Change the settings in ``/etc/nova/nova.conf`` as follows,
if they are not there by default.

.. code-block:: bash

    # /etc/nova/nova.conf
    [DEFAULT]
    security_group_api = neutron
    firewall_driver = nova.virt.firewall.NoopFirewallDriver

**OPNFV-NATIVE-SEC-3**: After updating the settings, you will have to restart the
``Neutron`` and ``Nova`` services.

**Please note that the commands of restarting** ``Neutron`` **and** ``Nova`` **would vary
depending on the installer. Please refer to relevant documentation of specific installers**

---------------------------------
Set Up Service VM as IPv6 vRouter
---------------------------------

**OPNFV-NATIVE-SETUP-1**: Now we assume that OpenStack multi-node setup is up and running.
We have to source the tenant credentials in OpenStack controller node in this step.
Please **NOTE** that the method of sourcing tenant credentials may vary depending on installers.
For example:

**Apex** installer:

.. code-block:: bash

    # On jump host, source the tenant credentials using /bin/opnfv-util provided by Apex installer
    opnfv-util undercloud "source overcloudrc; keystone service-list"

    # Alternatively, you can copy the file /home/stack/overcloudrc from the installer VM called "undercloud"
    # to a location in controller node, for example, in the directory /opt, and do:
    # source /opt/overcloudrc

**Compass** installer:

.. code-block:: bash

    # source the tenant credentials using Compass installer of OPNFV
    source /opt/admin-openrc.sh

**Fuel** installer:

.. code-block:: bash

    # source the tenant credentials using Fuel installer of OPNFV
    source /root/openrc

**Joid** installer:

.. code-block:: bash

    # source the tenant credentials using Joid installer of OPNFV
    source $HOME/joid_config/admin-openrc

**devstack**:

.. code-block:: bash

    # source the tenant credentials in devstack
    source openrc admin demo

**Please refer to relevant documentation of installers if you encounter any issue**.

**OPNFV-NATIVE-SETUP-2**: Download ``fedora22`` image which would be used for ``vRouter``

.. code-block:: bash

    wget https://download.fedoraproject.org/pub/fedora/linux/releases/22/Cloud/x86_64/\
    Images/Fedora-Cloud-Base-22-20150521.x86_64.qcow2

**OPNFV-NATIVE-SETUP-3**: Import Fedora22 image to ``glance``

.. code-block:: bash

    glance image-create --name 'Fedora22' --disk-format qcow2 --container-format bare \
    --file ./Fedora-Cloud-Base-22-20150521.x86_64.qcow2

**OPNFV-NATIVE-SETUP-4: This step is Informational. OPNFV Installer has taken care of this step
during deployment. You may refer to this step only if there is any issue, or if you are using other installers**.

We have to move the physical interface (i.e. the public network interface) to ``br-ex``, including moving
the public IP address and setting up default route. Please refer to ``OS-NATIVE-SETUP-4`` and
``OS-NATIVE-SETUP-5`` in our `more complete instruction <http://artifacts.opnfv.org/ipv6/docs/setupservicevm/5-ipv6-configguide-scenario-1-native-os.html#set-up-service-vm-as-ipv6-vrouter>`_.

**OPNFV-NATIVE-SETUP-5**: Create Neutron routers ``ipv4-router`` and ``ipv6-router``
which need to provide external connectivity.

.. code-block:: bash

    neutron router-create ipv4-router
    neutron router-create ipv6-router

**OPNFV-NATIVE-SETUP-6**: Create an external network/subnet ``ext-net`` using
the appropriate values based on the data-center physical network setup.

Please **NOTE** that you may only need to create the subnet of ``ext-net`` because OPNFV installers
should have created an external network during installation. You must use the same name of external
network that installer creates when you create the subnet. For example:

* **Apex** installer: ``external``
* **Compass** installer: ``ext-net``
* **Fuel** installer: ``admin_floating_net``
* **Joid** installer: ``ext-net``

**Please refer to the documentation of installers if there is any issue**

.. code-block:: bash

    # This is needed only if installer does not create an external work
    # Otherwise, skip this command "net-create"
    neutron net-create --router:external ext-net

    # Note that the name "ext-net" may work for some installers such as Compass and Joid
    # Change the name "ext-net" to match the name of external network that an installer creates
    neutron subnet-create --disable-dhcp --allocation-pool start=198.59.156.251,\
    end=198.59.156.254 --gateway 198.59.156.1 ext-net 198.59.156.0/24

**OPNFV-NATIVE-SETUP-7**: Create Neutron networks ``ipv4-int-network1`` and
``ipv6-int-network2`` with port_security disabled

.. code-block:: bash

    neutron net-create ipv4-int-network1
    neutron net-create ipv6-int-network2

**OPNFV-NATIVE-SETUP-8**: Create IPv4 subnet ``ipv4-int-subnet1`` in the internal network
``ipv4-int-network1``, and associate it to ``ipv4-router``.

.. code-block:: bash

    neutron subnet-create --name ipv4-int-subnet1 --dns-nameserver 8.8.8.8 \
    ipv4-int-network1 20.0.0.0/24

    neutron router-interface-add ipv4-router ipv4-int-subnet1

**OPNFV-NATIVE-SETUP-9**: Associate the ``ext-net`` to the Neutron routers ``ipv4-router``
and ``ipv6-router``.

.. code-block:: bash

    # Note that the name "ext-net" may work for some installers such as Compass and Joid
    # Change the name "ext-net" to match the name of external network that an installer creates
    neutron router-gateway-set ipv4-router ext-net
    neutron router-gateway-set ipv6-router ext-net

**OPNFV-NATIVE-SETUP-10**: Create two subnets, one IPv4 subnet ``ipv4-int-subnet2`` and
one IPv6 subnet ``ipv6-int-subnet2`` in ``ipv6-int-network2``, and associate both subnets to
``ipv6-router``

.. code-block:: bash

    neutron subnet-create --name ipv4-int-subnet2 --dns-nameserver 8.8.8.8 \
    ipv6-int-network2 10.0.0.0/24

    neutron subnet-create --name ipv6-int-subnet2 --ip-version 6 --ipv6-ra-mode slaac \
    --ipv6-address-mode slaac ipv6-int-network2 2001:db8:0:1::/64

    neutron router-interface-add ipv6-router ipv4-int-subnet2
    neutron router-interface-add ipv6-router ipv6-int-subnet2

**OPNFV-NATIVE-SETUP-11**: Create a keypair

.. code-block:: bash

    nova keypair-add vRouterKey > ~/vRouterKey

**OPNFV-NATIVE-SETUP-12**: Create ports for vRouter (with some specific MAC address
- basically for automation - to know the IPv6 addresses that would be assigned to the port).

.. code-block:: bash

    neutron port-create --name eth0-vRouter --mac-address fa:16:3e:11:11:11 ipv6-int-network2
    neutron port-create --name eth1-vRouter --mac-address fa:16:3e:22:22:22 ipv4-int-network1

**OPNFV-NATIVE-SETUP-13**: Create ports for VM1 and VM2.

.. code-block:: bash

    neutron port-create --name eth0-VM1 --mac-address fa:16:3e:33:33:33 ipv4-int-network1
    neutron port-create --name eth0-VM2 --mac-address fa:16:3e:44:44:44 ipv4-int-network1

**OPNFV-NATIVE-SETUP-14**: Update ``ipv6-router`` with routing information to subnet
``2001:db8:0:2::/64``

.. code-block:: bash

    neutron router-update ipv6-router --routes type=dict list=true \
    destination=2001:db8:0:2::/64,nexthop=2001:db8:0:1:f816:3eff:fe11:1111

**OPNFV-NATIVE-SETUP-15**: Boot Service VM (``vRouter``), VM1 and VM2

.. code-block:: bash

    nova boot --image Fedora22 --flavor m1.small \
    --user-data /opt/stack/opnfv_os_ipv6_poc/metadata.txt \
    --availability-zone nova:opnfv-os-compute \
    --nic port-id=$(neutron port-list | grep -w eth0-vRouter | awk '{print $2}') \
    --nic port-id=$(neutron port-list | grep -w eth1-vRouter | awk '{print $2}') \
    --key-name vRouterKey vRouter

    nova list

    # Please wait for some 10 to 15 minutes so that necessary packages (like radvd)
    # are installed and vRouter is up.
    nova console-log vRouter

    nova boot --image cirros-0.3.4-x86_64-uec --flavor m1.tiny \
    --user-data /opt/stack/opnfv_os_ipv6_poc/set_mtu.sh \
    --availability-zone nova:opnfv-os-controller \
    --nic port-id=$(neutron port-list | grep -w eth0-VM1 | awk '{print $2}') \
    --key-name vRouterKey VM1

    nova boot --image cirros-0.3.4-x86_64-uec --flavor m1.tiny
    --user-data /opt/stack/opnfv_os_ipv6_poc/set_mtu.sh \
    --availability-zone nova:opnfv-os-compute \
    --nic port-id=$(neutron port-list | grep -w eth0-VM2 | awk '{print $2}') \
    --key-name vRouterKey VM2

    nova list # Verify that all the VMs are in ACTIVE state.

**OPNFV-NATIVE-SETUP-16**: If all goes well, the IPv6 addresses assigned to the VMs
would be as shown as follows:

.. code-block:: bash

    # vRouter eth0 interface would have the following IPv6 address:
    #     2001:db8:0:1:f816:3eff:fe11:1111/64
    # vRouter eth1 interface would have the following IPv6 address:
    #     2001:db8:0:2::1/64
    # VM1 would have the following IPv6 address:
    #     2001:db8:0:2:f816:3eff:fe33:3333/64
    # VM2 would have the following IPv6 address:
    #     2001:db8:0:2:f816:3eff:fe44:4444/64

**OPNFV-NATIVE-SETUP-17**: Now we need to disable ``eth0-VM1``, ``eth0-VM2``,
``eth0-vRouter`` and ``eth1-vRouter`` port-security

.. code-block:: bash

    for port in eth0-VM1 eth0-VM2 eth0-vRouter eth1-vRouter
    do
        neutron port-update --no-security-groups $port
        neutron port-update $port --port-security-enabled=False
        neutron port-show $port | grep port_security_enabled
    done

**OPNFV-NATIVE-SETUP-18**: Now we can ``SSH`` to VMs. You can execute the following command.

.. code-block:: bash

    # 1. Create a floatingip and associate it with VM1, VM2 and vRouter (to the port id that is passed).
    #    Note that the name "ext-net" may work for some installers such as Compass and Joid
    #    Change the name "ext-net" to match the name of external network that an installer creates
    neutron floatingip-create --port-id $(neutron port-list | grep -w eth0-VM1 | \
    awk '{print $2}') ext-net
    neutron floatingip-create --port-id $(neutron port-list | grep -w eth0-VM2 | \
    awk '{print $2}') ext-net
    neutron floatingip-create --port-id $(neutron port-list | grep -w eth1-vRouter | \
    awk '{print $2}') ext-net

    # 2. To know / display the floatingip associated with VM1, VM2 and vRouter.
    neutron floatingip-list -F floating_ip_address -F port_id | grep $(neutron port-list | \
    grep -w eth0-VM1 | awk '{print $2}') | awk '{print $2}'
    neutron floatingip-list -F floating_ip_address -F port_id | grep $(neutron port-list | \
    grep -w eth0-VM2 | awk '{print $2}') | awk '{print $2}'
    neutron floatingip-list -F floating_ip_address -F port_id | grep $(neutron port-list | \
    grep -w eth1-vRouter | awk '{print $2}') | awk '{print $2}'

    # 3. To ssh to the vRouter, VM1 and VM2, user can execute the following command.
    ssh -i ~/vRouterKey fedora@<floating-ip-of-vRouter>
    ssh -i ~/vRouterKey cirros@<floating-ip-of-VM1>
    ssh -i ~/vRouterKey cirros@<floating-ip-of-VM2>

If everything goes well, ``ssh`` will be successful and you will be logged into those VMs.
Run some commands to verify that IPv6 addresses are configured on ``eth0`` interface.

**OPNFV-NATIVE-SETUP-19**: Show an IPv6 address with a prefix of ``2001:db8:0:2::/64``

.. code-block:: bash

    ip address show

**OPNFV-NATIVE-SETUP-20**: ping some external IPv6 address, e.g. ``ipv6-router``

.. code-block:: bash

    ping6 2001:db8:0:1::1

If the above ping6 command succeeds, it implies that ``vRouter`` was able to successfully forward the IPv6 traffic
to reach external ``ipv6-router``.

