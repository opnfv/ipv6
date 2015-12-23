=================================
Setting Up OpenStack Compute Node
=================================

For exemplary purpose, we assume:

* The hostname of OpenStack Compute Node is ``opnfv-os-compute``
* Ubuntu 14.04 is installed
* We use ``opnfv`` as username to login.
* We use ``devstack`` to install OpenStack Kilo

**OS-M-1**: Login to OpenStack Compute Node with username ``opnfv``

**OS-M-2**: Update the packages and install git

   ``sudo apt-get update -y``

   ``sudo apt-get install -y git``

**OS-M-3**: Download devstack and switch to stable/kilo branch

   ``git clone https://github.com/openstack-dev/devstack.git -b stable/kilo``

**OS-M-4**: Start a new terminal, and change directory to where OpenStack is installed.

   ``cd ~/devstack``

**OS-M-5**: Create a ``local.conf`` file with the contents from the following URL.

   ``http://fpaste.org/276958/44395955/``

   **Note 1**: you need to change the IP address of ``SERVICE_HOST`` to point to your actual IP address of OpenStack Controller.

   **Note 2**: you need to change the IP address of ``ODL_MGR_IP`` to point to your actual IP address of Open Daylight Controller.

   **Note 3**: You may have to change the value of ``ODL_PROVIDER_MAPPINGS`` and ``PUBLIC_INTERFACE`` to match your actual network interface.

**OS-M-6**: Initiate Openstack setup by invoking ``stack.sh``

   ``./stack.sh``

**OS-M-7**: Assuming that all goes well, you can set ``OFFLINE=True`` and ``RECLONE=no`` in ``local.conf`` to lock the codebase. Devstack uses these configuration parameters to determine if it has to run with the existing codebase or
update to the latest copy.

**OS-M-8**: Source the credentials.

   ``opnfv@opnfv-os-controller:~/devstack$ source openrc admin demo``

**OS-M-9**: Verify some commands to check if setup is working fine.

    ``opnfv@opnfv-os-controller:~/devstack$ nova flavor-list``
|    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
|    | ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public |
|    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
|    | 1  | m1.tiny   | 512       | 1    | 0         |      | 1     | 1.0         | True      |
|    | 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      |
|    | 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      |
|    | 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      |
|    | 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      |
|    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+

Now you can start to set up the service VM as an IPv6 vRouter in the environment of OpenStack and Open Daylight.
