.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Bin Hu (AT&T) and Sridhar Gaddam (RedHat)

========================================
Setting Up Open Daylight Controller Node
========================================

For exemplary purpose, we assume:

* The hostname of Open Daylight Controller Node is ``opnfv-odl-controller``, and the host IP address is
  ``192.168.0.30``
* CentOS 7 is installed
* We use ``opnfv`` as username to login.
* Java 7 is installed in directory ``/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.85-2.6.1.2.el7_1.x86_64/``

Please **NOTE** that this Scenario 3 uses an Open Daylight Lithium controller which is built
from the latest ``stable/Lithium`` branch that includes the fix of a bug, there is a **prerequisite**
that you are able to build this Open Daylight Lithium Controller from the  the latest ``stable/Lithium``
branch. Please refer to relevant documentation from Open Daylight.

**ODL-1**: **Prerequisite** - build Open Daylight Lithium Controller from the the latest
``stable/Lithium`` branch, and make it available for step **ODL-3**.

**ODL-2**: Login to Open Daylight Controller Node with username ``opnfv``.

**ODL-3**: Extract the tar file of your custom build of Open Daylight Lithium Controller
from step **ODL-1**.

.. code-block:: bash

    tar -zxvf <filename-of-your-custom-build>.tar.gz

**ODL-4**: Install Java7

.. code-block:: bash

    sudo yum install -y java-1.7.0-openjdk.x86_64

**ODL-5 (OPTIONAL)**: We are using ``iptables`` instead of
``firewalld`` but this is optional for the OpenDaylight Controller
Node. The objective is to allow all connections on the internal
private network (ens160). The same objective can be achieved using
firewalld as well. **If you intend to use firewalld, please skip this step and directly go to next step**:

.. code-block:: bash

    sudo systemctl stop firewalld.service
    sudo yum remove -y firewalld
    sudo yum install -y iptables-services
    sudo touch /etc/sysconfig/iptables
    sudo systemctl enable iptables.service
    sudo systemctl start iptables.service
    sudo iptables -I INPUT 1 -i ens160 -j ACCEPT
    sudo iptables -I INPUT -m state --state NEW -p tcp --dport 8181 -j ACCEPT # For ODL DLUX UI
    sudo iptables-save > /etc/sysconfig/iptables

**ODL-6**: Open a screen session.

.. code-block:: bash

    screen -S ODL_Controller

**ODL-7**: In the new screen session, change directory to where Open
Daylight is installed. Here we use ``odl`` directory name and
``Lithium SR3`` installation as an example.

.. code-block:: bash

    cd ~/odl/distribution-karaf-0.3.3-Lithium-SR3/bin

**ODL-8**: Set the JAVA environment variables.

.. code-block:: bash

    export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.85-2.6.1.2.el7_1.x86_64/jre
    export PATH=$PATH:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.85-2.6.1.2.el7_1.x86_64/jre/bin

**ODL-9**: Run the ``karaf`` shell.

.. code-block:: bash

    ./karaf

**ODL-10**: You are now in the Karaf shell of Open Daylight. To explore the list of available features you can execute
``feature:list``. In order to enable Open Daylight with OpenStack, you have to load the ``odl-ovsdb-openstack``
feature.

.. code-block:: bash

    opendaylight-user@opnfv>feature:install odl-ovsdb-openstack

**ODL-11**: Verify that OVSDB feature is installed successfully.

.. code-block:: bash

    opendaylight-user@opnfv>feature:list -i | grep ovsdb
    odl-ovsdb-openstack | 1.1.1-Lithium-SR1       | x  | ovsdb-1.1.1-Lithium-SR1 | OpenDaylight :: OVSDB :: OpenStack Network Virtual
    odl-ovsdb-southbound-api  | 1.1.1-Lithium-SR1 | x  | odl-ovsdb-southbound-1.1.1-Lithium-SR1 | OpenDaylight :: southbound :: api
    odl-ovsdb-southbound-impl | 1.1.1-Lithium-SR1 | x  | odl-ovsdb-southbound-1.1.1-Lithium-SR1 | OpenDaylight :: southbound :: impl
    odl-ovsdb-southbound-impl-rest|1.1.1-Lithium-SR1 | x | odl-ovsdb-southbound-1.1.1-Lithium-SR1| OpenDaylight :: southbound :: impl :: REST
    odl-ovsdb-southbound-impl-ui  | 1.1.1-Lithium-SR1| x | odl-ovsdb-southbound-1.1.1-Lithium-SR1| OpenDaylight :: southbound :: impl :: UI
    opendaylight-user@opnfv>

**ODL-12**: To view the logs, you can use the following commands (or alternately the file data/log/karaf.log).

.. code-block:: bash

    opendaylight-user@opnfv>log:display
    opendaylight-user@opnfv>log:tail

**ODL-13**: To enable ODL DLUX UI, install the following features.
Then you can navigate to
``http://<opnfv-odl-controller IP address>:8181/index.html`` for DLUX
UI. The default user-name and password is ``admin/admin``.

.. code-block:: bash

    opendaylight-user@opnfv>feature:install odl-dlux-core

**ODL-14**: To exit out of screen session, please use the command ``CTRL+a`` followed by ``d``

**Note: Do not kill the screen session, it will terminate the ODL controller.**

At this moment, Open Daylight has been started successfully.
