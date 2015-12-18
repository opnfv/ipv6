==========================================
Setting Up a Service VM as an IPv6 vRouter
==========================================

After OPNFV Brahmaputra Release base platform has been successfully installed through previous chapters, there are 11
steps to set up a service VM as an IPv6 vRouter:

- `Step 1: Disable odl-l3 and Enable neutron-l3-agent`_

- `Step 2: Start Open Daylight`_

- `Step 3: Start Open Stack on Controller Node`_

- `Step 4: Start Open Stack on Compute Node`_

- `Step 5: Create External Network Connectivity ext-net`_

- `Step 6: Create IPv4 Subnet and Router with External Connectivity`_

- `Step 7: Create IPv6 Subnet and Router with External Connectivity`_

- `Step 8: Prepare Image, Metadata and Keypair for Service VM`_

- `Step 9: Boot Service VM (vRouter) and other VMs in IPv6 Subnet`_

- `Step 10: Spawn RADVD in vRouter`_

- `Step 11: Testing to Verify Setup Complete`_

Once the setup is complete, you can go to `Next Steps`_.

*****************************************************
_`Step 1: Disable odl-l3 and Enable neutron-l3-agent`
*****************************************************

This step is optional, and only needed if you didn't choose to enable neutron-l3-agent during previous installation of
OPNFV Brahmaputra Release.

If you have chosen to enable neutron-l3-agent during installation, please skip this step and directly go to
`Step 2: Start Open Daylight`_.

# Place holder for instructions of how to disable odl-l3 and enable neutron-l3-agent

******************************
_`Step 2: Start Open Daylight`
******************************

**Note: we assume that you have installed Open Daylight through OPNFV Installer in prior chapters. However, if Open Daylight is not installed, please go to \`\`http://www.opendaylight.org/downloads\`\` to download and install Open Daylight**

ODL-1: Login to Open Daylight Controller Node. For the purpose of example, we use ``opnfv`` as username of login, and
``opnfv-odl-controller`` as hostname of the Open Daylight Controller Node.

ODL-2: Start a new terminal session, and change directory to where Open Daylight is installed. Here we use ``odl``
directory name and ``Lithium SR2`` installation as an example.

   ``cd ~/odl/distribution-karaf-0.3.2-Lithium-SR2/bin``

ODL-3: Run the ``karaf`` shell. Please note that it is recommended to run the command in a "screen" session.

|   ``screen -S ODL_Controller``
|   ``./karaf``

ODL-4: You are now in the Karaf shell of Open Daylight. To explore the list of available features you can execute
``feature:list``. In order to enable Open Daylight with Open Stack, you have to load the ``odl-ovsdb-openstack``
feature.

   ``opendaylight-user@opnfv>feature:install odl-ovsdb-openstack``

ODL-5: Verify that OVSDB feature is installed successfully.

|    ``opendaylight-user@opnfv>feature:list -i | grep ovsdb``
|    odl-ovsdb-openstack | 1.1.1-Lithium-SR1       | x  | ovsdb-1.1.1-Lithium-SR1 | OpenDaylight :: OVSDB :: OpenStack Network Virtual
|    odl-ovsdb-southbound-api  | 1.1.1-Lithium-SR1 | x  | odl-ovsdb-southbound-1.1.1-Lithium-SR1 | OpenDaylight :: southbound :: api
|    odl-ovsdb-southbound-impl | 1.1.1-Lithium-SR1 | x  | odl-ovsdb-southbound-1.1.1-Lithium-SR1 | OpenDaylight :: southbound :: impl
|    odl-ovsdb-southbound-impl-rest|1.1.1-Lithium-SR1 | x | odl-ovsdb-southbound-1.1.1-Lithium-SR1| OpenDaylight :: southbound :: impl :: REST
|    odl-ovsdb-southbound-impl-ui  | 1.1.1-Lithium-SR1| x | odl-ovsdb-southbound-1.1.1-Lithium-SR1| OpenDaylight :: southbound :: impl :: UI
|    ``opendaylight-user@opnfv>``

ODL-6: To view the logs, you can use the following commands (or alternately the file data/log/karaf.log).

|    ``opendaylight-user@opnfv>log:display``
|    ``opendaylight-user@opnfv>log:tail``

ODL-7: To enable ODL DLUX UI, install the following features. Then you can navigate to
``http://<opnfv-odl-controller IP address>:8181/index.html`` for DLUX UI.
The default user-name and password is admin/admin.

    ``opendaylight-user@opnfv>feature:install odl-restconf odl-l2switch-switch odl-mdsal-apidocs odl-dlux-core``

ODL-8: To exit out of screen session, please use the command ``CTRL+a`` followed by ``d``

**Note: Do not kill the screen session, it will terminate the ODL controller.**

At this moment, Open Daylight has been started successfully.

**********************************************
_`Step 3: Start Open Stack on Controller Node`
**********************************************

OS-N-1: Login to Open Stack Controller Node. For the purpose of example, we use ``opnfv`` as username of login, and
``opnfv-os-controller`` as hostname of the Open Stack Controller Node.

OS-N-2: Start a new terminal, and change directory to where Open Stack is installed. Here we use ``devstack`` directory
name as an example.

   ``cd ~/devstack``

OS-N-3: Create a ``local.conf`` file with the contents from the following URL.

   ``http://fpaste.org/276949/39476214/``

Note 1: You need to change the value of ``BRANCH``, and all appearance of ``stable/kilo`` and related URL to point to
the actual branch of your upstream repository.

Note 2: you need to change the IP address of ``ODL_MGR_IP`` to point to your actual IP address of Open Daylight
Controller.

Note 3: You may have to change the value of ``ODL_PROVIDER_MAPPINGS`` and ``PUBLIC_INTERFACE`` to match your actual
network interfaces.

OS-N-4: Initiate Openstack setup by invoking ``stack.sh``

   ``./stack.sh``

OS-N-5: If the setup is successful you would see the following logs on the console. Please note that the IP addresses
are all for the purpose of example. Your IP addresses will match the ones assigned during the installation of OPNFV B
Release base platform in prior chapters.

|   ``This is your host ip: <opnfv-os-controller IP address>``
|   ``Horizon is now available at http://<opnfv-os-controller IP address>/``
|   ``Keystone is serving at <opnfv-os-controller IP address>/``
|   ``The default users are: admin and demo``
|   ``The password: password``

OS-N-6: Assuming that all goes well, you can set ``OFFLINE=True`` and ``RECLONE=no`` in ``local.conf`` to lock the
codebase. Devstack uses these configuration parameters to determine if it has to run with the existing codebase or
update to the latest copy.

OS-N-7: Source the credentials.

   ``opnfv@opnfv-os-controller:~/devstack$ source openrc admin demo``

OS-N-8: Verify some commands to check if setup is working fine.

|    ``opnfv@opnfv-os-controller:~/devstack$ nova flavor-list``
|    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
|    | ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public |
|    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
|    | 1  | m1.tiny   | 512       | 1    | 0         |      | 1     | 1.0         | True      |
|    | 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      |
|    | 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      |
|    | 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      |
|    | 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      |
|    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+

Now you can start the Compute node setup.

*******************************************
_`Step 4: Start Open Stack on Compute Node`
*******************************************

OS-M-1: Login to Open Stack Compute Node. For the purpose of example, we use ``opnfv`` as username of login, and
``opnfv-os-compute`` as hostname of the Open Stack Compute Node.

OS-M-2: Start a new terminal, and change directory to where Open Stack is installed. Here we use ``devstack``
directory name as an example.

   ``cd ~/devstack``

OS-M-3: Create a ``local.conf`` file with the contents from the following URL.

   ``http://fpaste.org/276958/44395955/``

Note 1: You need to change the value of ``BRANCH``, and all appearance of ``stable/kilo`` and related URL to point to
the actual branch of your upstream repository.

Note 2: you need to change the IP address of ``SERVICE_HOST`` to point to your actual IP address of Open Stack
Controller.

Note 3: you need to change the IP address of ``ODL_MGR_IP`` to point to your actual IP address of Open Daylight
Controller.

Note 4: You may have to change the value of ``ODL_PROVIDER_MAPPINGS`` and ``PUBLIC_INTERFACE`` to match your actual
network interface.

OS-M-4: Initiate Openstack setup by invoking ``stack.sh``

   ``./stack.sh``

OS-M-5: Assuming that all goes well, you can set ``OFFLINE=True`` and ``RECLONE=no`` in ``local.conf`` to lock the
codebase. Devstack uses these configuration parameters to determine if it has to run with the existing codebase or
update to the latest copy.

OS-M-6: Source the credentials.

   ``opnfv@opnfv-os-compute:~/devstack$ source openrc admin demo``

OS-M-7:Verify some commands to check if setup is working fine.

|    ``opnfv@opnfv-os-compute:~/devstack$ nova flavor-list``
|    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
|    | ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public |
|    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
|    | 1  | m1.tiny   | 512       | 1    | 0         |      | 1     | 1.0         | True      |
|    | 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      |
|    | 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      |
|    | 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      |
|    | 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      |
|    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+

Now you can start to set up the service VM as an Ipv6 vRouter in the environment of Open Stack and Open Daylight.

*******************************************************
_`Step 5: Create External Network Connectivity ext-net`
*******************************************************

# Place holder for instructions of how to create ext-net

*******************************************************************
_`Step 6: Create IPv4 Subnet and Router with External Connectivity`
*******************************************************************

# Place holder for instructions of how to create IPv4 subnet and router associated with ext-net

*******************************************************************
_`Step 7: Create IPv6 Subnet and Router with External Connectivity`
*******************************************************************

# Place holder for instructions of how to create IPv6 subnet and router associated with ext-net

*************************************************************
_`Step 8: Prepare Image, Metadata and Keypair for Service VM`
*************************************************************

# Place holder for instructions of how to get the image and prepare the metadata for service VM, and how to add keypairs

*****************************************************************
_`Step 9: Boot Service VM (vRouter) and other VMs in IPv6 Subnet`
*****************************************************************

# Place holder for instructions of how to boot the service VM named vRouter, and a couple of others in the same Ipv6
subnet for testing purpose

**********************************
_`Step 10: Spawn RADVD in vRouter`
**********************************

# Place holder for instructions of how to spawn the RADVD daemon in vRouter

********************************************
_`Step 11: Testing to Verify Setup Complete`
********************************************

# Place holder for instructions of how to test and verify that the setup is complete

*************
_`Next Steps`
*************

Congratulations, you have completed the setup of using a service VM to act as an IPv6 vRouter. This setup allows further
open innovation by any 3rd-party. Please refer to relevant sections in User's Guide for further value-added services on
this IPv6 vRouter.

