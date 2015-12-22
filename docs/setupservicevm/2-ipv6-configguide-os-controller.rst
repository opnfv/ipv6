===============================
Setting Up OpenStack Controller
===============================

For exemplary purpose, we assume:
* The hostname of OpenStack Controller Node is ``opnfv-os-
controller``
* Ubuntu 14.04 is installed
* We use ``opnfv`` as username to login.
* We use ``devstack`` to install OpenStack Kilo

**OS-N-1**: Login to OpenStack Controller Node with username ``opnfv``

**OS-N-2**: Update the packages and install git

|   ``sudo apt-get update -y``
|   ``sudo apt-get install -y git``

**OS-N-3**: Download devstack and switch to stable/kilo branch

   ``git clone https://github.com/openstack-dev/devstack.git -b stable/kilo``

**OS-N-4**: Start a new terminal, and change directory to where OpenStack is installed.

   ``cd ~/devstack``

**OS-N-5**: Create a ``local.conf`` file with the contents from the following URL.

   ``http://fpaste.org/276949/39476214/``

**Note 1**: you need to change the IP address of ``ODL_MGR_IP`` to
point to your actual IP address of Open Daylight Controller.

**Note 2**: You may have to change the value of
``ODL_PROVIDER_MAPPINGS`` and ``PUBLIC_INTERFACE`` to match your
actual network interfaces.

**OS-N-6**: Initiate Openstack setup by invoking ``stack.sh``

   ``./stack.sh``

**OS-N-7**: If the setup is successful you would see the following logs on the console. Please note that the IP addresses
are all for the purpose of example. Your IP addresses will match the ones assigned during the installation of OPNFV B
Release base platform in prior chapters.

|   ``This is your host ip: <opnfv-os-controller IP address>``
|   ``Horizon is now available at http://<opnfv-os-controller IP address>/``
|   ``Keystone is serving at http://<opnfv-os-controller IP address>:5000/``
|   ``The default users are: admin and demo``
|   ``The password: password``

**OS-N-8**: Assuming that all goes well, you can set ``OFFLINE=True`` and ``RECLONE=no`` in ``local.conf`` to lock the
codebase. Devstack uses these configuration parameters to determine if it has to run with the existing codebase or
update to the latest copy.

**OS-N-9**: Source the credentials.

   ``opnfv@opnfv-os-controller:~/devstack$ source openrc admin demo``

**OS-N-10**: Verify some commands to check if setup is working fine.

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
