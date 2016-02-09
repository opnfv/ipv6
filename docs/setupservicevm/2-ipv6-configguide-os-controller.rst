.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

====================================
Setting Up OpenStack Controller Node
====================================

Please **note** that the instructions shown here are using ``devstack`` installer. If you are an experienced
user and installs OpenStack in a different way, you can skip this step and follow the instructions of the
method you are using to install OpenStack.

For exemplary purpose, we assume:

* The hostname of OpenStack Controller Node is ``opnfv-os-controller``, and the host IP address is ``192.168.0.10``
* Ubuntu 14.04 or Fedora 21 is installed
* We use ``opnfv`` as username to login.
* We use ``devstack`` to install OpenStack Kilo. Please note that although the instructions are based on
  OpenStack Kilo, they can be applied to Liberty in the same way.

**OS-N-0**: Login to OpenStack Controller Node with username ``opnfv``

**OS-N-1**: Update the packages and install git

For **Ubuntu**:

.. code-block:: bash

    sudo apt-get update -y
    sudo apt-get install -y git

For **Fedora**:

.. code-block:: bash

    sudo yum update -y
    sudo yum install -y git

**OS-N-2**: Clone the following GitHub repository to get the configuration and metadata files

.. code-block:: bash

    git clone https://github.com/sridhargaddam/opnfv_os_ipv6_poc.git /opt/stack/opnfv_os_ipv6_poc

**OS-N-3**: Download devstack and switch to stable/kilo branch

.. code-block:: bash

    git clone https://github.com/openstack-dev/devstack.git -b stable/kilo

**OS-N-4**: Start a new terminal, and change directory to where OpenStack is installed.

.. code-block:: bash

    cd ~/devstack

**OS-N-5**: Create a ``local.conf`` file from the GitHub repo we cloned at **OS-N-2**.

.. code-block:: bash

    cp /opt/stack/opnfv_os_ipv6_poc/scenario2/local.conf.odl.controller ~/devstack/local.conf

Please **note** that you need to change the IP address of ``ODL_MGR_IP`` to point to your actual IP address
of Open Daylight Controller.

**OS-N-6**: Initiate Openstack setup by invoking ``stack.sh``

.. code-block:: bash

    ./stack.sh

**OS-N-7**: If the setup is successful you would see the following logs on the console. Please note
that the IP addresses are all for the purpose of example. Your IP addresses will match the ones
of your actual network interfaces.

.. code-block:: bash

    This is your host IP address: 192.168.0.10
    This is your host IPv6 address: ::1
    Horizon is now available at http://192.168.0.10/
    Keystone is serving at http://192.168.0.10:5000/
    The default users are: admin and demo
    The password: password

Please **note** that The IP addresses above are exemplary purpose. It will show you the actual IP address of your host.

**OS-N-8**: Assuming that all goes well, you can set ``OFFLINE=True`` and ``RECLONE=no`` in ``local.conf``
to lock the codebase. Devstack uses these configuration parameters to determine if it has to run with
the existing codebase or update to the latest copy.

**OS-N-9**: Source the credentials.

.. code-block:: bash

    opnfv@opnfv-os-controller:~/devstack$ source openrc admin demo

Please **NOTE** that the method of sourcing tenant credentials may vary depending on installers.
**Please refer to relevant documentation of installers if you encounter any issue**.

**OS-N-10**: Verify some commands to check if setup is working fine.

.. code-block:: bash

    opnfv@opnfv-os-controller:~/devstack$ nova flavor-list
    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
    | ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public |
    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
    | 1  | m1.tiny   | 512       | 1    | 0         |      | 1     | 1.0         | True      |
    | 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      |
    | 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      |
    | 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      |
    | 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      |
    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+

Now you can start the Compute node setup.
