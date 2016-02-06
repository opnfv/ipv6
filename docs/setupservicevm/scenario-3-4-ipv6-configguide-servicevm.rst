.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

==========================================
Setting Up a Service VM as an IPv6 vRouter
==========================================

Now we can start to set up a service VM as an IPv6 vRouter. For exemplary purpose, we assume:

* The hostname of  Open Daylight Controller Node is ``opnfv-odl-controller``, and the host IP address is
  ``192.168.0.30``
* The hostname of OpenStack Controller Node is ``opnfv-os-controller``, and the host IP address
  is ``192.168.0.10``
* The hostname of OpenStack Compute Node is ``opnfv-os-compute``, and the host IP address is ``192.168.0.20``
* We use ``opnfv`` as username to login.
* We use ``devstack`` to install OpenStack Kilo, and the directory is ``~/devstack``
* Note: all IP addresses as shown below are for exemplary purpose.

****************************************************
Note: Disable Security Groups in OpenStack ML2 Setup
****************************************************

Please note that Security Groups feature has been disabled automatically through ``local.conf`` configuration file
during the setup procedure of OpenStack in both
`Controller Node <./scenario-3-2-ipv6-configguide-os-controller.html>`_
and `Compute Node <./scenario-3-3-ipv6-configguide-os-compute.html>`_ using ``devstack``.

If you are installing OpenStack using a different installer (i.e. not with ``devstack``), please make sure
that Security Groups are disabled in the setup.

**Please refer to**
`here <./5-ipv6-configguide-scenario-1-native-os.html#note-disable-security-groups-in-openstack-ml2-setup>`_
**for the notes in** ``Section 2.4``, **steps** ``OS-NATIVE-SEC-1`` **through** ``OS-NATIVE-SEC-3``.

*********************************
Set Up Service VM as IPv6 vRouter
*********************************

**SCENARIO-3-SETUP-1**: Now we assume that OpenStack multi-node setup is up and running. The following
commands should be executed:

.. code-block:: bash

    cd ~/devstack

    # source the tenant credentials in devstack
    source openrc admin demo

Please **NOTE** that the method of sourcing tenant credentials may vary depending on installers.
**Please refer to relevant documentation of installers if you encounter any issue**.

**SCENARIO-3-SETUP-2**: Download ``fedora22`` image which would be used for ``vRouter``

.. code-block:: bash

    wget https://download.fedoraproject.org/pub/fedora/linux/releases/22/Cloud/x86_64/Images/Fedora-Cloud-Base-22-20150521.x86_64.qcow2

**SCENARIO-3-SETUP-3**: Import Fedora22 image to ``glance``

.. code-block:: bash

    glance image-create --name 'Fedora22' --disk-format qcow2 --container-format bare --file ./Fedora-Cloud-Base-22-20150521.x86_64.qcow2

**SCENARIO-3-SETUP-4**: Now we have to move the physical interface (i.e. the public network interface)
to ``br-ex``, including moving the public IP address and setting up default route. **Please note that this step
may already have been done when you use a different installer to deploy OpenStack because that installer
may have already moved the physical interface to** ``br-ex`` **during deployment**.

Because our ``opnfv-os-controller`` node has two interfaces ``eth0`` and ``eth1``,
and ``eth1`` is used for external connectivity, move the IP address of ``eth1`` to ``br-ex``.

Please note that the IP address ``198.59.156.113`` and related subnet and gateway addressed in the command
below are for exemplary purpose. **Please replace them with the IP addresses of your actual network**.

.. code-block:: bash

    sudo ip addr del 198.59.156.113/24 dev eth1
    sudo ovs-vsctl add-port br-ex eth1
    sudo ifconfig eth1 up
    sudo ip addr add 198.59.156.113/24 dev br-ex
    sudo ifconfig br-ex up
    sudo ip route add default via 198.59.156.1 dev br-ex

**SCENARIO-3-SETUP-5**: Verify that ``br-ex`` now has the original external IP address, and that the default route is on
``br-ex``

.. code-block:: bash

    opnfv@opnfv-os-controller:~/devstack$ ip a s br-ex
    38: br-ex: <BROADCAST,UP,LOWER_UP> mtu 1430 qdisc noqueue state UNKNOWN group default
        link/ether 00:50:56:82:42:d1 brd ff:ff:ff:ff:ff:ff
        inet 198.59.156.113/24 brd 198.59.156.255 scope global br-ex
           valid_lft forever preferred_lft forever
        inet6 fe80::543e:28ff:fe70:4426/64 scope link
           valid_lft forever preferred_lft forever
    opnfv@opnfv-os-controller:~/devstack$
    opnfv@opnfv-os-controller:~/devstack$ ip route
    default via 198.59.156.1 dev br-ex
    192.168.0.0/24 dev eth0  proto kernel  scope link  src 192.168.0.10
    192.168.122.0/24 dev virbr0  proto kernel  scope link  src 192.168.122.1
    198.59.156.0/24 dev br-ex  proto kernel  scope link  src 198.59.156.113

Please note that the IP addresses above are exemplary purpose.

**SCENARIO-3-SETUP-6**: Create Neutron routers ``ipv4-router`` and ``ipv6-router`` which need to provide external
connectivity.

.. code-block:: bash

    neutron router-create ipv4-router
    neutron router-create ipv6-router

**SCENARIO-3-SETUP-7**: Create an external network/subnet ``ext-net`` using the appropriate values based on the
data-center physical network setup.

Please **NOTE** that if you use a different installer, i.e. NOT ``devstack``, your installer
may have already created an external network during installation. Under this circumstance,
you may only need to create the subnet of ``ext-net``. When you create the subnet, you must
use the same name of external network that your installer creates.

.. code-block:: bash

    # If you use a different installer and it has already created an external work,
    # Please skip this command "net-create"
    neutron net-create --router:external ext-net

    # If you use a different installer and it has already created an external work,
    # Change the name "ext-net" to match the name of external network that your installer has created
    neutron subnet-create --disable-dhcp --allocation-pool start=198.59.156.251,end=198.59.156.254 --gateway 198.59.156.1 ext-net 198.59.156.0/24

**SCENARIO-3-SETUP-8**: Create Neutron networks ``ipv4-int-network1`` and ``ipv6-int-network2``

.. code-block:: bash

    neutron net-create ipv4-int-network1
    neutron net-create ipv6-int-network2

**SCENARIO-3-SETUP-9**: Create IPv4 subnet ``ipv4-int-subnet1`` in the internal network ``ipv4-int-network1``,
and associate it to ``ipv4-router``.

.. code-block:: bash

    neutron subnet-create --name ipv4-int-subnet1 --dns-nameserver 8.8.8.8 ipv4-int-network1 20.0.0.0/24
    neutron router-interface-add ipv4-router ipv4-int-subnet1

**SCENARIO-3-SETUP-10**: Associate the ``ext-net`` to the Neutron routers ``ipv4-router`` and ``ipv6-router``.

.. code-block:: bash

    # If you use a different installer and it has already created an external work,
    # Change the name "ext-net" to match the name of external network that your installer has created
    neutron router-gateway-set ipv4-router ext-net
    neutron router-gateway-set ipv6-router ext-net

**SCENARIO-3-SETUP-11**: Create two subnets, one IPv4 subnet ``ipv4-int-subnet2`` and one IPv6 subnet
``ipv6-int-subnet2`` in ``ipv6-int-network2``, and associate both subnets to ``ipv6-router``

.. code-block:: bash

    neutron subnet-create --name ipv4-int-subnet2 --dns-nameserver 8.8.8.8 ipv6-int-network2 10.0.0.0/24
    neutron subnet-create --name ipv6-int-subnet2 --ip-version 6 --ipv6-ra-mode slaac --ipv6-address-mode slaac ipv6-int-network2 2001:db8:0:1::/64
    neutron router-interface-add ipv6-router ipv4-int-subnet2
    neutron router-interface-add ipv6-router ipv6-int-subnet2

**SCENARIO-3-SETUP-12**: Create a keypair

.. code-block:: bash

    nova keypair-add vRouterKey > ~/vRouterKey

**SCENARIO-3-SETUP-13**: Create ports for vRouter (with some specific MAC address - basically for automation -
to know the IPv6 addresses that would be assigned to the port).

.. code-block:: bash

    neutron port-create --name eth0-vRouter --mac-address fa:16:3e:11:11:11 ipv6-int-network2
    neutron port-create --name eth1-vRouter --mac-address fa:16:3e:22:22:22 ipv4-int-network1

**SCENARIO-3-SETUP-14**: Create ports for VM1 and VM2.

.. code-block:: bash

    neutron port-create --name eth0-VM1 --mac-address fa:16:3e:33:33:33 ipv4-int-network1
    neutron port-create --name eth0-VM2 --mac-address fa:16:3e:44:44:44 ipv4-int-network1

**SCENARIO-3-SETUP-15**: Update ``ipv6-router`` with routing information to subnet ``2001:db8:0:2::/64``

.. code-block:: bash

    neutron router-update ipv6-router --routes type=dict list=true destination=2001:db8:0:2::/64,nexthop=2001:db8:0:1:f816:3eff:fe11:1111

**SCENARIO-3-SETUP-16**: Boot Service VM (``vRouter``), VM1 and VM2

.. code-block:: bash

    nova boot --image Fedora22 --flavor m1.small --user-data /opt/stack/opnfv_os_ipv6_poc/metadata.txt --availability-zone nova:opnfv-os-compute --nic port-id=$(neutron port-list | grep -w eth0-vRouter | awk '{print $2}') --nic port-id=$(neutron port-list | grep -w eth1-vRouter | awk '{print $2}') --key-name vRouterKey vRouter
    nova list
    nova console-log vRouter #Please wait for some 10 to 15 minutes so that necessary packages (like radvd) are installed and vRouter is up.
    nova boot --image cirros-0.3.4-x86_64-uec --flavor m1.tiny --nic port-id=$(neutron port-list | grep -w eth0-VM1 | awk '{print $2}') --availability-zone nova:opnfv-os-controller --key-name vRouterKey --user-data /opt/stack/opnfv_os_ipv6_poc/set_mtu.sh VM1
    nova boot --image cirros-0.3.4-x86_64-uec --flavor m1.tiny --nic port-id=$(neutron port-list | grep -w eth0-VM2 | awk '{print $2}') --availability-zone nova:opnfv-os-compute --key-name vRouterKey --user-data /opt/stack/opnfv_os_ipv6_poc/set_mtu.sh VM2
    nova list # Verify that all the VMs are in ACTIVE state.

**SCENARIO-3-SETUP-17**: If all goes well, the IPv6 addresses assigned to the VMs would be as shown as follows:

.. code-block:: bash

    vRouter eth0 interface would have the following IPv6 address: 2001:db8:0:1:f816:3eff:fe11:1111/64
    vRouter eth1 interface would have the following IPv6 address: 2001:db8:0:2::1/64
    VM1 would have the following IPv6 address: 2001:db8:0:2:f816:3eff:fe33:3333/64
    VM2 would have the following IPv6 address: 2001:db8:0:2:f816:3eff:fe44:4444/64

**SCENARIO-3-SETUP-18**: Now we can ``SSH`` to VMs. You can execute the following command.

.. code-block:: bash

    # 1. Create a floatingip and associate it with VM1, VM2 and vRouter (to the port id that is passed).
    #    If you use a different installer and it has already created an external work,
    #    Change the name "ext-net" to match the name of external network that your installer has created
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

