.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

=================================
Setting Up OpenStack Compute Node
=================================

Please **note** that the instructions shown here are using ``devstack`` installer. If you are an experienced user
and installs OpenStack in a different way, you can skip this step and follow the instructions of the method you
are using to install OpenStack.

For exemplary purpose, we assume:

* The hostname of OpenStack Compute Node is ``opnfv-os-compute``, and the host IP address is ``192.168.0.20``
* Ubuntu 14.04 or Fedora 21 is installed
* We use ``opnfv`` as username to login.
* We use ``devstack`` to install OpenStack Kilo. Please note that although the instructions are based on
  OpenStack Kilo, they can be applied to Liberty in the same way.

**OS-M-0**: Login to OpenStack Compute Node with username ``opnfv``

**OS-M-1**: Update the packages and install git

For **Ubuntu**:

.. code-block:: bash

    sudo apt-get update -y
    sudo apt-get install -y git

For **Fedora**:

.. code-block:: bash

    sudo yum update -y
    sudo yum install -y git

**OS-M-2**: Clone the following GitHub repository to get the configuration and metadata files

.. code-block:: bash

    git clone https://github.com/sridhargaddam/opnfv_os_ipv6_poc.git /opt/stack/opnfv_os_ipv6_poc

**OS-M-3**: Download devstack and switch to stable/kilo branch

.. code-block:: bash

    git clone https://github.com/openstack-dev/devstack.git -b stable/kilo

**OS-M-4**: Start a new terminal, and change directory to where OpenStack is installed.

.. code-block:: bash

    cd ~/devstack

**OS-M-5**: Create a ``local.conf`` file from the GitHub repo we cloned at **OS-M-2**.

.. code-block:: bash

    cp /opt/stack/opnfv_os_ipv6_poc/scenario2/local.conf.odl.compute ~/devstack/local.conf

Please Note:

* Note 1: you need to change the IP address of ``SERVICE_HOST`` to point to your actual IP address
  of OpenStack Controller.
* Note 2: you need to change the IP address of ``ODL_MGR_IP`` to point to your actual IP address
  of Open Daylight Controller.

**OS-M-6**: Initiate Openstack setup by invoking ``stack.sh``

.. code-block:: bash

    ./stack.sh

**OS-M-7**: Assuming that all goes well, you should see the following output.

.. code-block:: bash

    This is your host IP address: 192.168.0.20
    This is your host IPv6 address: ::1

Please **note** that The IP addresses above are exemplary purpose. It will show you the actual IP address of your host.

You can set ``OFFLINE=True`` and ``RECLONE=no`` in ``local.conf`` to lock the codebase. Devstack uses these
configuration parameters to determine if it has to run with the existing codebase or update to the latest copy.

**OS-M-8**: Source the credentials.

.. code-block:: bash

    opnfv@opnfv-os-compute:~/devstack$ source openrc admin demo

Please **NOTE** that the method of sourcing tenant credentials may vary depending on installers.
**Please refer to relevant documentation of installers if you encounter any issue**.

**OS-M-9**: You can verify that OpenStack is set up correctly by showing hypervisor list

.. code-block:: bash

    opnfv@opnfv-os-compute:~/devstack$ nova hypervisor-list
    +----+------------------------------------+---------+------------+
    | ID  | Hypervisor hostname      | State  | Status  |
    +----+------------------------------------+---------+------------+
    | 1   | opnfv-os-controller      | up     | enabled |
    | 2   | opnfv-os-compute         | up     | enabled |
    +----+------------------------------------+---------+------------+

Now you can start to set up the service VM as an IPv6 vRouter in the environment of OpenStack and Open Daylight.
