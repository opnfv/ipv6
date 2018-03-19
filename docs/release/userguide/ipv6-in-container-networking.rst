.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Prakash Ramchandran

======================================
Exploring IPv6 in Container Networking
======================================

This document is the summary of how to use IPv6 with Docker.

The defualt Docker container uses 172.17.0.0/24 subnet with 172.17.0.1 as gateway.
So IPv6 network needs to be enabled and configured before we can use it with IPv6
traffic.

We will describe how to use IPv6 in Docker in the following 5 sections:

1. Install Docker Community Edition (CE)
2. IPv6 with Docker
3. Design Simple IPv6 Topologies
4. Design Solutions
5. Challenges in Production Use

-------------------------------------
Install Docker Community Edition (CE)
-------------------------------------

**Step 3.1.1**: Download Docker (CE) on your system from "this link" [1]_.

For Ubuntu 16.04 Xenial x86_64, please refer to "Docker CE for Ubuntu" [2]_.

**Step 3.1.2**: Refer to "this link" [3]_ to install Docker CE on Xenial.

**Step 3.1.3**: Once you installed the docker, you can verify the standalone
default bridge nework as follows:

.. code-block:: bash

    $ docker network ls
    NETWORK ID NAME DRIVER SCOPE
    b9e92f9a8390 bridge bridge local
    74160ae686b9 host host local
    898fbb0a0c83 my_bridge bridge local
    57ac095fdaab none null local

Note that:

* the details may be different with different network drivers.
* User-defined bridge networks are the best when you need multiple containers
  to communicate on the same Docker host.
* Host networks are the best when the network stack should not be isolated from
  the Docker host, but you want other aspects of the container to be isolated.
* Overlay networks are the best when you need containers running on different
  Docker hosts to communicate, or when multiple applications work together
  using swarm services.
* Macvlan networks are the best when you are migrating from a VM setup or need
  your containers to look like physical hosts on your network, each with a
  unique MAC address.
* Third-party network plugins allow you to integrate Docker with specialized
  network stacks. Please refer to "Docker Networking Tutorials" [4]_.

.. code-block:: bash

    # This will have docker0 default bridge details showing
    # ipv4 172.17.0.1/16 and
    # ipv6 fe80::42:4dff:fe2f:baa6/64 entries

    $ ip addr show
    11: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:4d:2f:ba:a6 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
    valid_lft forever preferred_lft forever
    inet6 fe80::42:4dff:fe2f:baa6/64 scope link
    valid_lft forever preferred_lft forever

Thus we see here a simple defult ipv4 networking for docker. Inspect and verify
that IPv6 address is not listed here showing its enabled but not used by
default docker0 bridge.

You can create user defined bridge network using command like ``my_bridge``
below with other than default, e.g. 172.18.0.0/24 here. **Note** that ``--ipv6``
is not specified yet

.. code-block:: bash

    $ sudo docker network create \
                  --driver=bridge \
                  --subnet=172.18.0.0/24 \
                  --gaeway= 172.18.0.1 \
                  my_bridge

    $ docker network inspect bridge
    [
      {
        "Name": "bridge",
        "Id": "b9e92f9a839048aab887081876fc214f78e8ce566ef5777303c3ef2cd63ba712",
        "Created": "2017-10-30T23:32:15.676301893-07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "ea76bd4694a8073b195dd712dd0b070e80a90e97b6e2024b03b711839f4a3546": {
            "Name": "registry",
            "EndpointID": "b04dc6c5d18e3bf4e4201aa8ad2f6ad54a9e2ea48174604029576e136b99c49d",
            "MacAddress": "02:42:ac:11:00:02",
            "IPv4Address": "172.17.0.2/16",
            "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
      }
    ]

    $ sudo docker network inspect my_bridge
    [
      {
        "Name": "my_bridge",
        "Id": "898fbb0a0c83acc0593897f5af23b1fe680d38b804b0d5a4818a4117ac36498a",
        "Created": "2017-07-16T17:59:55.388151772-07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
      }
    ]

You can note that IPv6 is not enabled here yet as seen through network inspect.
Since we have only IPv4 installed with Docker, we will move to enable IPv6 for
Docker in the next step.

----------------
IPv6 with Docker
----------------

Verifyig IPv6 with Docker involves the following steps:

**Step 3.2.1**: Enable ipv6 support for Docker

In the simplest term, the first step is to enable IPv6 on Docker on Linux hosts.
Please refer to "this link" [5]_:

* Edit ``/etc/docker/daemon.json``
* Set the ``ipv6`` key to true.

.. code-block:: bash

    {{{ "ipv6": true }}}

Save the file.

**Step 3.2.1.1**: Set up IPv6 addressing for Docker in ``daemon.json``

If you need IPv6 support for Docker containers, you need to enable the option
on the Docker daemon ``daemon.json`` and reload its configuration, before
creating any IPv6 networks or assigning containers IPv6 addresses.

When you create your network, you can specify the ``--ipv6`` flag to enable
IPv6. You can't selectively disable IPv6 support on the default bridge network.

**Step 3.2.1.2**: Enable forwarding from Docker containers to the outside world

By default, traffic from containers connected to the default bridge network is
not forwarded to the outside world. To enable forwarding, you need to change
two settings. These are not Docker commands and they affect the Docker host's
kernel.

* Setting 1: Configure the Linux kernel to allow IP forwarding:

.. code-block:: bash

    $ sysctl net.ipv4.conf.all.forwarding=1

* Setting 2: Change the policy for the iptables FORWARD policy from DROP to ACCEPT.

.. code-block:: bash

    $ sudo iptables -P FORWARD ACCEPT

These settings do not persist across a reboot, so you may need to add them to
a start-up script.

**Step 3.2.1.3**: Use the default bridge network

The default bridge network is considered a legacy detail of Docker and is not
recommended for production use. Configuring it is a manual operation, and it
has technical shortcomings.

**Step 3.2.1.4**: Connect a container to the default bridge network

If you do not specify a network using the ``--network`` flag, and you do
specify a network driver, your container is connected to the default bridge
network by default. Containers connected to the default bridge network can
communicate, but only by IP address, unless they are linked using the legacy
``--link`` flag.

**Step 3.2.1.5**: Configure the default bridge network

To configure the default bridge network, you specify options in ``daemon.json``.
Here is an example of ``daemon.json`` with several options specified. Only
specify the settings you need to customize.

.. code-block:: bash

    {
      "bip": "192.168.1.5/24",
      "fixed-cidr": "192.168.1.5/25",
      "fixed-cidr-v6": "2001:db8::/64",
      "mtu": 1500,
      "default-gateway": "10.20.1.1",
      "default-gateway-v6": "2001:db8:abcd::89",
      "dns": ["10.20.1.2","10.20.1.3"]
    }

Restart Docker for the changes to take effect.

**Step 3.2.1.6**: Use IPv6 with the default bridge network

If you configure Docker for IPv6 support (see **Step 2.1.1**), the default
bridge network is also configured for IPv6 automatically. Unlike user-defined
bridges, you cannot selectively disable IPv6 on the default bridge.

**Step 3.2.1.7**: Reload the Docker configuration file

.. code-block:: bash

    $ systemctl reload docker

**Step 3.2.1.8**: You can now create networks with the ``--ipv6`` flag and assign
containers IPv6 addresses.

**Step 3.2.1.9**: Verify your host and docker networks

.. code-block:: bash

    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    ea76bd4694a8        registry:2          "/entrypoint.sh /e..."   x months ago        Up y months         0.0.0.0:4000->5000/tcp   registry

    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    b9e92f9a8390        bridge              bridge              local
    74160ae686b9        host                host                local
    898fbb0a0c83        my_bridge           bridge              local
    57ac095fdaab        none                null                local

**Step 3.2.1.10**: Edit ``/etc/docker/daemon.json`` and set the ipv6 key to true.

.. code-block:: bash

    {
      "ipv6": true
    }

Save the file.

**Step 3.2.1.11**: Reload the Docker configuration file.

.. code-block:: bash

    $ sudo systemctl reload docker

**Step 3.2.1.12**: You can now create networks with the ``--ipv6`` flag and
assign containers IPv6 addresses using the ``--ip6`` flag.

.. code-block:: bash

    $ sudo docker network create --ipv6 --driver bridge alpine-net--fixed-cidr-v6 2001:db8:1/64

    # "docker network create" requires exactly 1 argument(s).
    # See "docker network create --help"

Earlier, user was allowed to create a network, or start the daemon, without
specifying an IPv6 ``--subnet``, or ``--fixed-cidr-v6`` respectively, even when
using the default builtin IPAM driver, which does not support auto allocation
of IPv6 pools. In another word, it was an incorrect configurations, which had
no effect on IPv6 stuff. It was a no-op.

A fix cleared that so that Docker will now correctly consult with the IPAM
driver to acquire an IPv6 subnet for the bridge network, when user did not
supply one.

If the IPAM driver in use is not able to provide one, network creation would
fail (in this case the default bridge network).

So what you see now is the expected behavior. You need to remove the ``--ipv6``
flag when you start the daemon, unless you pass a ``--fixed-cidr-v6`` pool. We
should probably clarify this somewhere.

The above was found on following Docker.

.. code-block:: bash

    $ docker info
    Containers: 27
    Running: 1
    Paused: 0
    Stopped: 26
    Images: 852
    Server Version: 17.06.1-ce-rc1
    Storage Driver: aufs
      Root Dir: /var/lib/docker/aufs
      Backing Filesystem: extfs
      Dirs: 637
      Dirperm1 Supported: false
    Logging Driver: json-file
    Cgroup Driver: cgroupfs
    Plugins:
      Volume: local
      Network: bridge host macvlan null overlay
      Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
    Swarm: inactive
    Runtimes: runc
    Default Runtime: runc
    Init Binary: docker-init
    containerd version: 6e23458c129b551d5c9871e5174f6b1b7f6d1170
    runc version: 810190ceaa507aa2727d7ae6f4790c76ec150bd2
    init version: 949e6fa
    Security Options:
      apparmor
      seccomp
      Profile: default
    Kernel Version: 3.13.0-88-generic
    Operating System: Ubuntu 16.04.2 LTS
    OSType: linux
    Architecture: x86_64
    CPUs: 4
    Total Memory: 11.67GiB
    Name: aatiksh
    ID: HS5N:T7SK:73MD:NZGR:RJ2G:R76T:NJBR:U5EJ:KP5N:Q3VO:6M2O:62CJ
    Docker Root Dir: /var/lib/docker
    Debug Mode (client): false
    Debug Mode (server): false
    Registry: https://index.docker.io/v1/
    Experimental: false
    Insecure Registries:
      127.0.0.0/8
    Live Restore Enabled: false

**Step 3.2.2**: Check the network drivers

Among the 4 supported drivers, we will be using "User-Defined Bridge Network" [6]_.

-----------------------------
Design Simple IPv6 Topologies
-----------------------------

**Step 3.3.1**: Creating IPv6 user-defined subnet.

Let's create a Docker with IPv6 subnet:

.. code-block:: bash

    $ sudo docker network create \
                  --ipv6 \
                  --driver=bridge \
                  --subnet=172.18.0.0/16 \
                  --subnet=fcdd:1::/48 \
                  --gaeway= 172.20.0.1  \
                  my_ipv6_bridge

    # Error response from daemon:
    
    cannot create network 8957e7881762bbb4b66c3e2102d72b1dc791de37f2cafbaff42bdbf891b54cc3 (br-8957e7881762): conflicts with network
    no matching subnet for range 2002:ac14:0000::/48

    # try changing to ip-addess-range instead of subnet for ipv6.
    # networks have overlapping IPv4

    NETWORK ID          NAME                DRIVER              SCOPE
    b9e92f9a8390        bridge              bridge              local
    74160ae686b9        host                host                local
    898fbb0a0c83        my_bridge           bridge              local
    57ac095fdaab        none                null                local
    no matching subnet for gateway 172.20.01

    # So finally making both as subnet and gateway as 172.20.0.1 works

    $ sudo docker network create \
                  --ipv6 \
                  --driver=bridge \
                  --subnet=172.20.0.0/16 \
                  --subnet=2002:ac14:0000::/48 \
                  --gateway=172.20.0.1 \
                  my_ipv6_bridge
    898fbb0a0c83acc0593897f5af23b1fe680d38b804b0d5a4818a4117ac36498a (br-898fbb0a0c83):

Since lxdbridge used the ip range on the system there was a conflict.
This brings us to question how do we assign IPv6 and IPv6 address for our solutions.

----------------
Design Solutions
----------------

For best practices, please refer to "Best Practice Document" [7]_.

Use IPv6 Calcualtor at "this link" [8]_.

* For IPv4 172.16.0.1   = 6to4 prefix 2002:ac10:0001::/48
* For IPv4 172.17.01/24 = 6to4 prefix 2002:ac11:0001::/48
* For IPv4 172.18.0.1   = 6to4 prefix 2002:ac12:0001::/48
* For IPv4 172.19.0.1   = 6to4 prefix 2002:ac13:0001::/48
* For IPv4 172.20.0.0   = 6to4 prefix 2002:ac14:0000::/48

To avoid overlaping IP's, let's use the .20 in our design:

.. code-block:: bash

    $ sudo docker network create \
                  --ipv6 \
                  --driver=bridge \
                  --subnet=172.20.0.0/24 \
                  --subnet=2002:ac14:0000::/48
                  --gateway=172.20.0.1
                  my_ipv6_bridge

    # created ...

    052da268171ce47685fcdb68951d6d14e70b9099012bac410c663eb2532a0c87

    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    b9e92f9a8390        bridge              bridge              local
    74160ae686b9        host                host                local
    898fbb0a0c83        my_bridge           bridge              local
    052da268171c        my_ipv6_bridge      bridge              local
    57ac095fdaab        none                null                local

    # Note the first 16 digits is used here as network id from what we got
    # whaen we created it.

    $ docker network  inspect my_ipv6_bridge
    [
      {
        "Name": "my_ipv6_bridge",
        "Id": "052da268171ce47685fcdb68951d6d14e70b9099012bac410c663eb2532a0c87",
        "Created": "2018-03-16T07:20:17.714212288-07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": true,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
                },
                {
                    "Subnet": "2002:ac14:0000::/48"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
      }
    ]

Note that:

* IPv6 flag is ebnabled and that IPv6 range is listed besides Ipv4 gateway.
* We are mapping IPv4 and IPv6 address to simplify assignments as per "Best
  Pratice Document" [7]_.

Testing the solution and topology:

.. code-block:: bash

    $ sudo docker run hello-world
    Hello from Docker!

This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:

1. The Docker client contacted the Docker daemon.
2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
3. The Docker daemon created a new container from that image which runs the
   executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it
   to your terminal.

To try something more ambitious, you can run an Ubuntu container with:

.. code-block:: bash

    $ docker run -it ubuntu bash

    root@62b88b030f5a:/# ls
    bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
    boot  etc  lib   media  opt  root  sbin  sys  usr

On terminal it appears that the docker is functioning normally.

Let's now push to see if we can use the ``my_ipv6_bridge`` network.
Please refer to "User-Defined Bridge Network" [9]_.

++++++++++++++++++++++++++++++++++++++++++++
Connect a container to a user-defined bridge
++++++++++++++++++++++++++++++++++++++++++++

When you create a new container, you can specify one or more ``--network``
flags. This example connects a Nginx container to the ``my-net`` network. It
also publishes port 80 in the container to port 8080 on the Docker host, so
external clients can access that port. Any other container connected to the
``my-net`` network has access to all ports on the my-nginx container, and vice
versa.

.. code-block:: bash

    $ docker create --name my-nginx \
                    --network my-net \
                    --publish 8080:80 \
                    nginx:latest

To connect a running container to an existing user-defined bridge, use the
``docker network connect`` command. The following command connects an
already-running ``my-nginx`` container to an already-existing ``my_ipv6_bridge``
network:

.. code-block:: bash

    $ docker network connect my_ipv6_bridge my-nginx

Now we have connected the IPv6-enabled network to ``mynginx`` conatiner. Let's
start and verify its IP Address:

.. code-block:: bash

    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    df1df6ed3efb        alpine              "ash"                    4 hours ago         Up 4 hours                                   alpine1
    ea76bd4694a8        registry:2          "/entrypoint.sh /e..."   9 months ago        Up 4 months         0.0.0.0:4000->5000/tcp   registry

The ``nginx:latest`` image is not runnung, so let's start and log into it.

.. code-block:: bash

    $ docker images | grep latest
    REPOSITORY                                          TAG                 IMAGE ID            CREATED             SIZE
    nginx                                               latest              73acd1f0cfad        2 days ago          109MB
    alpine                                              latest              3fd9065eaf02        2 months ago        4.15MB
    swaggerapi/swagger-ui                               latest              e0b4f5dd40f9        4 months ago        23.6MB
    ubuntu                                              latest              d355ed3537e9        8 months ago        119MB
    hello-world                                         latest              1815c82652c0        9 months ago        1.84kB

Now we do find the ``nginx`` and let`s run it

.. code-block:: bash

    $ docker run -i -t nginx:latest /bin/bash
    root@bc13944d22e1:/# ls
    bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
    boot  etc  lib   media  opt  root  sbin  sys  usr
    root@bc13944d22e1:/#

Open another terminal and check the networks and verify that IPv6 address is
listed on the container:

.. code-block:: bash

    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
    bc13944d22e1        nginx:latest        "/bin/bash"              About a minute ago   Up About a minute   80/tcp                   loving_hawking
    df1df6ed3efb        alpine              "ash"                    4 hours ago          Up 4 hours                                   alpine1
    ea76bd4694a8        registry:2          "/entrypoint.sh /e..."   9 months ago         Up 4 months         0.0.0.0:4000->5000/tcp   registry

    $ ping6 bc13944d22e1

    # On 2nd termoinal

    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    b9e92f9a8390        bridge              bridge              local
    74160ae686b9        host                host                local
    898fbb0a0c83        my_bridge           bridge              local
    052da268171c        my_ipv6_bridge      bridge              local
    57ac095fdaab        none                null                local

    $ ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 8c:dc:d4:6e:d5:4b brd ff:ff:ff:ff:ff:ff
        inet 10.0.0.80/24 brd 10.0.0.255 scope global dynamic eno1
           valid_lft 558367sec preferred_lft 558367sec
        inet6 2601:647:4001:739c:b80a:6292:1786:b26/128 scope global dynamic
           valid_lft 86398sec preferred_lft 86398sec
        inet6 fe80::8edc:d4ff:fe6e:d54b/64 scope link
           valid_lft forever preferred_lft forever
    11: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:4d:2f:ba:a6 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.1/16 scope global docker0
           valid_lft forever preferred_lft forever
        inet6 fe80::42:4dff:fe2f:baa6/64 scope link
           valid_lft forever preferred_lft forever
    20: br-052da268171c: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:5e:19:55:0d brd ff:ff:ff:ff:ff:ff
        inet 172.20.0.1/16 scope global br-052da268171c
           valid_lft forever preferred_lft forever
        inet6 2002:ac14::1/48 scope global
           valid_lft forever preferred_lft forever
        inet6 fe80::42:5eff:fe19:550d/64 scope link
           valid_lft forever preferred_lft forever
        inet6 fe80::1/64 scope link
           valid_lft forever preferred_lft forever

Note that on the 20th entry we have the ``br-052da268171c`` with IPv6
``inet6 2002:ac14::1/48`` scope global, which belongs to root@bc13944d22e1.

At this time we have been able to provide a simple Docker with IPv6 solution.

+++++++++++++++++++++++++++++++++++++++++++++++++
Disconnect a container from a user-defined bridge
+++++++++++++++++++++++++++++++++++++++++++++++++

If another route needs to be added to ``nginx``, you need to modify the routes:

.. code-block:: bash

    # using ip route commands

    $ ip r
    default via 10.0.0.1 dev eno1  proto static  metric 100
    default via 10.0.0.1 dev wlan0  proto static  metric 600
    10.0.0.0/24 dev eno1  proto kernel  scope link  src 10.0.0.80
    10.0.0.0/24 dev wlan0  proto kernel  scope link  src 10.0.0.38
    10.0.0.0/24 dev eno1  proto kernel  scope link  src 10.0.0.80  metric 100
    10.0.0.0/24 dev wlan0  proto kernel  scope link  src 10.0.0.38  metric 600
    10.0.8.0/24 dev lxdbr0  proto kernel  scope link  src 10.0.8.1
    169.254.0.0/16 dev lxdbr0  scope link  metric 1000
    172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.0.1
    172.18.0.0/16 dev br-898fbb0a0c83  proto kernel  scope link  src 172.18.0.1
    172.20.0.0/16 dev br-052da268171c  proto kernel  scope link  src 172.20.0.1
    192.168.99.0/24 dev vboxnet1  proto kernel  scope link  src 192.168.99.1

If the routes are correctly updated you should be able to see ``nginx`` web
page on link ``http://172.20.0.0.1``

We now have completed the exercise.

To disconnect a running container from a user-defined bridge, use the
``docker network disconnect`` command. The following command disconnects the
``my-nginx`` container from the ``my-net`` network.

.. code-block:: bash

    $ docker network disconnect my_ipv6_bridge my-nginx

The IPv6 Docker we used is for demo purpose only. For real production we need
to follow one of the IPv6 solutions we have come across.

----------------------------
Challenges in Production Use
----------------------------

"This link" [10]_ discusses the details of the use of ``nftables`` which
is nextgen ``iptables``, and tries to build production worthy Docker for IPv6
usage.

----------
References
----------

.. [1] https://www.docker.com/community-edition#/download
.. [2] https://store.docker.com/editions/community/docker-ce-server-ubuntu
.. [3] https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1
.. [4] https://docs.docker.com/network/network-tutorial-host/#other-networking-tutorials
.. [5] https://docs.docker.com/config/daemon/ipv6/
.. [6] https://docs.docker.com/network/
.. [7] https://networkengineering.stackexchange.com/questions/119/ipv6-address-space-layout-best-practices
.. [8] http://www.gestioip.net/cgi-bin/subnet_calculator.cgi
.. [9] https://docs.docker.com/network/bridge/#use-ipv6-with-the-default-bridge-network
.. [10] https://stephank.nl/p/2017-06-05-ipv6-on-production-docker.html
