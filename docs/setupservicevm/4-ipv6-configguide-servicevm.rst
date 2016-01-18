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
during the setup procedure of OpenStack in both `Controller Node <./2-ipv6-configguide-os-controller.html>`_
and `Compute Node <./3-ipv6-configguide-os-compute.html>`_ using ``devstack``.

If you are installing OpenStack using a different installer (i.e. not with ``devstack``), please make sure
that Security Groups are disabled in the setup.

**Please refer to
`here <./5-ipv6-configguide-scenario-1-native-os.html#note-disable-security-groups-in-openstack-ml2-setup>`_
for the notes in** ``Section 2.4``, **steps** ``OS-NATIVE-SEC-1`` **through** ``OS-NATIVE-SEC-3``.

***************************************************
Source the Credentials in OpenStack Controller Node
***************************************************

**SETUP-SVM-1**: Login with username ``opnfv`` in OpenStack Controller Node ``opnfv-os-controller``.
Start a new terminal, and change directory to where OpenStack is installed.

.. code-block:: bash

    cd ~/devstack

**SETUP-SVM-2**: Source the credentials.

.. code-block:: bash

    # source the tenant credentials in devstack
    opnfv@opnfv-os-controller:~/devstack$ source openrc admin demo

Please **NOTE** that the method of sourcing tenant credentials may vary depending on installers.
**Please refer to relevant documentation of installers if you encounter any issue**.

**************************************
Add External Connectivity to ``br-ex``
**************************************

Because we need to manually create networks/subnets to achieve the IPv6 vRouter, we have used the flag
``NEUTRON_CREATE_INITIAL_NETWORKS=False`` in ``local.conf`` file. When this flag is set to False,
``devstack`` does not create any networks/subnets during the setup phase.

Now we have to move the physical interface (i.e. the public network interface) to ``br-ex``,
including moving the public IP address and setting up default route. **Please note that this step
may already have been done when you use a different installer to deploy OpenStack because that installer
may have already moved the physical interface to** ``br-ex`` **during deployment**.

In OpenStack Controller Node ``opnfv-os-controller``, ``eth1`` is configured to provide external/public connectivity
for both IPv4 and IPv6 (optional). So let us add this interface to ``br-ex`` and move the IP address, including the
default route from ``eth1`` to ``br-ex``.

**SETUP-SVM-3**: Add ``eth1`` to ``br-ex`` and move the IP address and the default route from ``eth1`` to ``br-ex``

.. code-block:: bash

    sudo ip addr del 198.59.156.113/24 dev eth1
    sudo ovs-vsctl add-port br-ex eth1
    sudo ifconfig eth1 up
    sudo ip addr add 198.59.156.113/24 dev br-ex
    sudo ifconfig br-ex up
    sudo ip route add default via 198.59.156.1 dev br-ex

Please note that:

* The IP address ``198.59.156.113`` and related subnet and gateway addressed in the command
  below are for exemplary purpose. **Please replace them with the IP addresses of your actual network**.
* **This can be automated in /etc/network/interfaces**.

**SETUP-SVM-4**: Verify that ``br-ex`` now has the original external IP address, and that the default route is on
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

Please  note that The IP addresses above are exemplary purpose

********************************************************
Create IPv4 Subnet and Router with External Connectivity
********************************************************

**SETUP-SVM-5**: Create a Neutron router ``ipv4-router`` which needs to provide external connectivity.

.. code-block:: bash

    neutron router-create ipv4-router

**SETUP-SVM-6**: Create an external network/subnet ``ext-net`` using the appropriate values based on the
data-center physical network setup.

.. code-block:: bash

    neutron net-create --router:external ext-net
    neutron subnet-create --disable-dhcp --allocation-pool start=198.59.156.251,end=198.59.156.254 --gateway 198.59.156.1 ext-net 198.59.156.0/24

Please note that the IP addresses in the command above are for exemplary purpose. **Please replace the IP addresses of
your actual network**.

**SETUP-SVM-7**: Associate the ``ext-net`` to the Neutron router ``ipv4-router``.

.. code-block:: bash

    neutron router-gateway-set ipv4-router ext-net

**SETUP-SVM-8**: Create an internal/tenant IPv4 network ``ipv4-int-network1``

.. code-block:: bash

    neutron net-create ipv4-int-network1

**SETUP-SVM-9**: Create an IPv4 subnet ``ipv4-int-subnet1`` in the internal network ``ipv4-int-network1``

.. code-block:: bash

    neutron subnet-create --name ipv4-int-subnet1 --dns-nameserver 8.8.8.8 ipv4-int-network1 20.0.0.0/24

**SETUP-SVM-10**: Associate the IPv4 internal subnet ``ipv4-int-subnet1`` to the Neutron router ``ipv4-router``.

.. code-block:: bash

    neutron router-interface-add ipv4-router ipv4-int-subnet1

********************************************************
Create IPv6 Subnet and Router with External Connectivity
********************************************************

Now, let us create a second neutron router where we can "manually" spawn a ``radvd`` daemon to simulate an external
IPv6 router.

**SETUP-SVM-11**:  Create a second Neutron router ``ipv6-router`` which needs to provide external connectivity

.. code-block:: bash

    neutron router-create ipv6-router

**SETUP-SVM-12**: Associate the ``ext-net`` to the Neutron router ``ipv6-router``

.. code-block:: bash

    neutron router-gateway-set ipv6-router ext-net

**SETUP-SVM-13**: Create a second internal/tenant IPv4 network ``ipv4-int-network2``

.. code-block:: bash

    neutron net-create ipv4-int-network2

**SETUP-SVM-14**: Create an IPv4 subnet ``ipv4-int-subnet2`` for the ``ipv6-router`` internal network
``ipv4-int-network2``

.. code-block:: bash

    neutron subnet-create --name ipv4-int-subnet2 --dns-nameserver 8.8.8.8 ipv4-int-network2 10.0.0.0/24

**SETUP-SVM-15**: Associate the IPv4 internal subnet ``ipv4-int-subnet2`` to the Neutron router ``ipv6-router``.

.. code-block:: bash

    neutron router-interface-add ipv6-router ipv4-int-subnet2

**************************************************
Prepare Image, Metadata and Keypair for Service VM
**************************************************

**SETUP-SVM-16**: Download ``fedora22`` image which would be used as ``vRouter``

.. code-block:: bash

    glance image-create --name 'Fedora22' --disk-format qcow2 --container-format bare --is-public true --copy-from https://download.fedoraproject.org/pub/fedora/linux/releases/22/Cloud/x86_64/Images/Fedora-Cloud-Base-22-20150521.x86_64.qcow2

**SETUP-SVM-17**: Create a keypair

.. code-block:: bash

    nova keypair-add vRouterKey > ~/vRouterKey

**SETUP-SVM-18**: Create ports for ``vRouter`` and both the VMs with some specific MAC addresses.

.. code-block:: bash

    neutron port-create --name eth0-vRouter --mac-address fa:16:3e:11:11:11 ipv4-int-network2
    neutron port-create --name eth1-vRouter --mac-address fa:16:3e:22:22:22 ipv4-int-network1
    neutron port-create --name eth0-VM1 --mac-address fa:16:3e:33:33:33 ipv4-int-network1
    neutron port-create --name eth0-VM2 --mac-address fa:16:3e:44:44:44 ipv4-int-network1

**********************************************************************************************************
Boot Service VM (``vRouter``) with ``eth0`` on ``ipv4-int-network2`` and ``eth1`` on ``ipv4-int-network1``
**********************************************************************************************************

Let us boot the service VM (``vRouter``) with ``eth0`` interface on ``ipv4-int-network2`` connecting to ``ipv6-router``,
and ``eth1`` interface on ``ipv4-int-network1`` connecting to ``ipv4-router``.

**SETUP-SVM-19**: Boot the ``vRouter`` using ``Fedora22`` image on the OpenStack Compute Node with hostname
``opnfv-os-compute``

.. code-block:: bash

    nova boot --image Fedora22 --flavor m1.small --user-data /opt/stack/opnfv_os_ipv6_poc/metadata.txt --availability-zone nova:opnfv-os-compute --nic port-id=$(neutron port-list | grep -w eth0-vRouter | awk '{print $2}') --nic port-id=$(neutron port-list | grep -w eth1-vRouter | awk '{print $2}') --key-name vRouterKey vRouter

Please **note** that ``/opt/stack/opnfv_os_ipv6_poc/metadata.txt`` is used to enable the ``vRouter`` to automatically
spawn a ``radvd``, and

* Act as an IPv6 vRouter which advertises the RA (Router Advertisements) with prefix
  ``2001:db8:0:2::/64`` on its internal interface (``eth1``).
* Forward IPv6 traffic from internal interface (``eth1``)

**SETUP-SVM-20**: Verify that ``Fedora22`` image boots up successfully and vRouter has ``ssh`` keys properly injected

.. code-block:: bash

    nova list
    nova console-log vRouter

Please note that **it may take a few minutes** for the necessary packages to get installed and ``ssh`` keys
to be injected.

.. code-block:: bash

    # Sample Output
    [  762.884523] cloud-init[871]: ec2: #############################################################
    [  762.909634] cloud-init[871]: ec2: -----BEGIN SSH HOST KEY FINGERPRINTS-----
    [  762.931626] cloud-init[871]: ec2: 2048 e3:dc:3d:4a:bc:b6:b0:77:75:a1:70:a3:d0:2a:47:a9   (RSA)
    [  762.957380] cloud-init[871]: ec2: -----END SSH HOST KEY FINGERPRINTS-----
    [  762.979554] cloud-init[871]: ec2: #############################################################

*******************************************
Boot Two Other VMs in ``ipv4-int-network1``
*******************************************

In order to verify that the setup is working, let us create two cirros VMs with ``eth1`` interface on the
``ipv4-int-network1``, i.e., connecting to ``vRouter`` ``eth1`` interface for internal network.

We will have to configure appropriate ``mtu`` on the VMs' interface by taking into account the tunneling
overhead and any physical switch requirements. If so, push the ``mtu`` to the VM either using ``dhcp``
options or via ``meta-data``.

**SETUP-SVM-21**: Create VM1 on OpenStack Controller Node with hostname ``opnfv-os-controller``

.. code-block:: bash

    nova boot --image cirros-0.3.4-x86_64-uec --flavor m1.tiny --nic port-id=$(neutron port-list | grep -w eth0-VM1 | awk '{print $2}') --availability-zone nova:opnfv-os-controller --key-name vRouterKey --user-data /opt/stack/opnfv_os_ipv6_poc/set_mtu.sh VM1

**SETUP-SVM-22**: Create VM2 on OpenStack Compute Node with hostname ``opnfv-os-compute``

.. code-block:: bash

    nova boot --image cirros-0.3.4-x86_64-uec --flavor m1.tiny --nic port-id=$(neutron port-list | grep -w eth0-VM2 | awk '{print $2}') --availability-zone nova:opnfv-os-compute --key-name vRouterKey --user-data /opt/stack/opnfv_os_ipv6_poc/set_mtu.sh VM2

**SETUP-SVM-23**: Confirm that both the VMs are successfully booted.

.. code-block:: bash

    nova list
    nova console-log VM1
    nova console-log VM2

**********************************
Spawn ``RADVD`` in ``ipv6-router``
**********************************

Let us manually spawn a ``radvd`` daemon inside ``ipv6-router`` namespace to simulate an external router.
First of all, we will have to identify the ``ipv6-router`` namespace and move to the namespace.

Please **NOTE** that in case of HA (High Availability) deployment model where multiple controller
nodes are used, ``ipv6-router`` created in step **SETUP-SVM-11** could be in any of the controller
node. Thus you need to identify in which controller node ``ipv6-router`` is created in order to manually
spawn ``radvd`` daemon inside the ``ipv6-router`` namespace in steps **SETUP-SVM-24** through
**SETUP-SVM-30**. The following command in Neutron will display the controller on which the
``ipv6-router`` is spawned.

.. code-block:: bash

    neutron l3-agent-list-hosting-router ipv6-router

Then you login to that controller and execute steps **SETUP-SVM-24**
through **SETUP-SVM-30**

**SETUP-SVM-24**: identify the ``ipv6-router`` namespace and move to the namespace

.. code-block:: bash

    sudo ip netns exec qrouter-$(neutron router-list | grep -w ipv6-router | awk '{print $2}') bash

**SETUP-SVM-25**: Upon successful execution of the above command, you will be in the router namespace.
Now let us configure the IPv6 address on the <qr-xxx> interface.

.. code-block:: bash

    export router_interface=$(ip a s | grep -w "global qr-*" | awk '{print $7}')
    ip -6 addr add 2001:db8:0:1::1 dev $router_interface

**SETUP-SVM-26**: Update the sample file ``/opt/stack/opnfv_os_ipv6_poc/scenario2/radvd.conf``
with ``$router_interface``.

.. code-block:: bash

    cp /opt/stack/opnfv_os_ipv6_poc/scenario2/radvd.conf /tmp/radvd.$router_interface.conf
    sed -i 's/$router_interface/'$router_interface'/g' /tmp/radvd.$router_interface.conf

**SETUP-SVM-27**: Spawn a ``radvd`` daemon to simulate an external router. This ``radvd`` daemon advertises an IPv6
subnet prefix of ``2001:db8:0:1::/64`` using RA (Router Advertisement) on its $router_interface so that ``eth0``
interface of ``vRouter`` automatically configures an IPv6 SLAAC address.

.. code-block:: bash

    $radvd -C /tmp/radvd.$router_interface.conf -p /tmp/br-ex.pid.radvd -m syslog

**SETUP-SVM-28**: Add an IPv6 downstream route pointing to the ``eth0`` interface of vRouter.

.. code-block:: bash

    ip -6 route add 2001:db8:0:2::/64 via 2001:db8:0:1:f816:3eff:fe11:1111

**SETUP-SVM-29**: The routing table should now look similar to something shown below.

.. code-block:: bash

    ip -6 route show
    2001:db8:0:1::1 dev qr-42968b9e-62 proto kernel metric 256
    2001:db8:0:1::/64 dev qr-42968b9e-62 proto kernel metric 256 expires 86384sec
    2001:db8:0:2::/64 via 2001:db8:0:1:f816:3eff:fe11:1111 dev qr-42968b9e-62 proto ra metric 1024 expires 29sec
    fe80::/64 dev qg-3736e0c7-7c proto kernel metric 256
    fe80::/64 dev qr-42968b9e-62 proto kernel metric 256

**SETUP-SVM-30**: If all goes well, the IPv6 addresses assigned to the VMs would be as shown as follows:

.. code-block:: bash

    vRouter eth0 interface would have the following IPv6 address: 2001:db8:0:1:f816:3eff:fe11:1111/64
    vRouter eth1 interface would have the following IPv6 address: 2001:db8:0:2::1/64
    VM1 would have the following IPv6 address: 2001:db8:0:2:f816:3eff:fe33:3333/64
    VM2 would have the following IPv6 address: 2001:db8:0:2:f816:3eff:fe44:4444/64

********************************
Testing to Verify Setup Complete
********************************

Now, let us ``ssh`` to one of the VMs, e.g. VM1, to confirm that it has successfully configured the IPv6 address
using ``SLAAC`` with prefix ``2001:db8:0:2::/64`` from ``vRouter``.

Please note that you need to get the IPv4 address associated to VM1. This can be inferred from ``nova list`` command.

**SETUP-SVM-31**: ``ssh`` VM1

.. code-block:: bash

    ssh -i /home/odl/vRouterKey cirros@<VM1-IPv4-address>

If everything goes well, ``ssh`` will be successful and you will be logged into VM1. Run some commands to verify
that IPv6 addresses are configured on ``eth0`` interface.

**SETUP-SVM-32**: Show an IPv6 address with a prefix of ``2001:db8:0:2::/64``

.. code-block:: bash

    ip address show

**SETUP-SVM-33**: ping some external IPv6 address, e.g. ``ipv6-router``

.. code-block:: bash

    ping6 2001:db8:0:1::1

If the above ping6 command succeeds, it implies that ``vRouter`` was able to successfully forward the IPv6 traffic
to reach external ``ipv6-router``.

**SETUP-SVM-34**: When all tests show that the setup works as expected, You can now exit the ``ipv6-router`` namespace.

.. code-block:: bash

    exit

**********
Next Steps
**********

Congratulations, you have completed the setup of using a service VM to act as an IPv6 vRouter. This setup allows further
open innovation by any 3rd-party. Please refer to relevant sections in User's Guide for further value-added services on
this IPv6 vRouter.

