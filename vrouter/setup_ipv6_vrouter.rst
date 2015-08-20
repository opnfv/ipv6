======================================
Set up an IPv6 vRouter on a Service VM
======================================

| Here you will find the steps involved in creating a ServiceVM that acts as an IPv6 vRouter. In this example, we will be using a CentOS7 image as vRouter (we should be able to use other OS as well) and devstack for OpenStack installation. We need to enable Port Security Extension as the extension_drivers in ML2 configuration file.

| Following is a sample configuration of devstack local.conf file.

| **# [[local|localrc]]**
|   `DATA_DIR=$DEST/data`
|   `SCREEN_LOGDIR=$DATA_DIR/logs`
|   `LOGFILE=$SCREEN_LOGDIR/stack.sh.log`
|   `ADMIN_PASSWORD=password`
|   `MYSQL_PASSWORD=password`
|   `RABBIT_PASSWORD=password`
|   `SERVICE_PASSWORD=password`
|   `SERVICE_TOKEN=token`
|   `disable_service n-net tempest h-eng h-api h-api-cfn h-api-cw`
|   `enable_service q-svc q-dhcp q-meta q-agt q-l3 n-novnc`
| **# [[post-config|/$Q_PLUGIN_CONF_FILE]]**
| **# [ml2]**
|   `extension_drivers=port_security`

| After successful installation of OpenStack with the above configuration, we shall create the necessary neutron networks/subnets/ports etc.
|   `cd devstack`
|   `./stack.sh`

| # Source the tenant credentials.
|   `source openrc admin demo`
| # Create a Neutron router which provides external connectivity.
|   `neutron router-create router1`
| # Create an external network using the appropriate values based on the data-center physical network setup.
|   `neutron net-create --provider:network_type <flat/vlan> --provider:physical_network <physical-network> --provider:segmentation_id <segmentation-id-if-vlan> --router:external ext-net`
| # Configure ipv6_gateway=<LLA-of-upstream-router> in the Neutron L3 agent configuration file.
| # Associate the ext-net to the neutron router.
|   `neutron router-gateway-set router1 ext-net`
| # Create an IPv6 internal network.
|   `neutron net-create ipv6-internal-network`
| # Create an IPv6 subnet in the internal network.
|   `neutron subnet-create --name ipv6-int-subnet --ip-version 6 --ipv6-ra-mode slaac --ipv6-address-mode slaac ipv6-internal-network 2001:db8:0:1::/64`
| # Associate the internal subnet to a neutron router. 
|   `neutron router-interface-add router1 ipv6-int-subnet`
   
| Now we shall create an isolated network which is the internal network of vRouter.
| # Create an isolated router for the tenant internal network.
|   `neutron router-create router2`
| # Create a Neutron Internal Network.
|   `neutron net-create tenant-internal-network`
| # Create an IPv4 subnet in the internal network.
|   `neutron subnet-create --name ipv4-int-subnet tenant-internal-network 10.0.0.1/24`
| # Associate the router2 to IPv4 subnet created above.
|   `neutron router-interface-add <router2-id> <ipv4-int-subnet-id>`

| Mapping this configuration to `PoC-1`_.

.. _`PoC-1`: /ipv6/images/ipv6-poc-1.png

- `ipv6-internal-network and ext-net is the Red colored network.`
- `tenant-internal-network is the Green colored network.`

| Lets create two neutron ports one from ext-net and the other from tenant-internal-network for the vRouter VM
|   `neutron port-create ipv6-internal-network --port-security-enabled=False --name enp0s3-port`
|   `neutron port-create tenant-internal-network --port-security-enabled=False --name enp0s8-port`

| Download the Centos7 image which is used as vRouter.
|   `glance image-create --name 'Centos7' --disk-format qcow2 --container-format bare --is-public true --copy-from http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2`

| Create a keypair.
|   `nova keypair-add vRouterKey > ~/vRouterKey`

| Spawn the Centos7 image with two nics (i.e., enp0s3-port and enp0s8-port)
|   `nova boot --image <Centos7-image-id> –flavor m1.small --nic port-id=$(neutron port-show -f value -F id enp0s3-port) –nic --nic port-id=$(neutron port-show -f value -F id enp0s8-port) --key-name vRouterKey CentOSvRouter`
  
| Verify that CentOSvRouter boots up successfully and keypair is injected.
|   `nova list`
|   `nova console-log CentOSvRouter`

| After the image boots up successfully, from the router1 namespace, ssh to vRouter using the keypair.
|   `sudo ip netns`
|   `sudo ip netns exec <router1-namespace> bash`
|   `ssh -i ~/vRouterKey centos@<ip-address-of-the-image>`
  
| As a one time job, before we can create the snapshot, execute the steps (i.e., SLAAC setup) mentioned at the following link.
|   `https://wiki.opnfv.org/ipv6_opnfv_project/vm_as_router`

| In order to verify that the setup is working, lets create some cirros VMs on the "tenant-internal-network" (i.e., vRouter internal network).
|   `nova boot --image <Cirros-image-id> --flavor m1.tiny --nic net-id=<tenant-internal-network-id> VM1`
|   `nova boot --image <Cirros-image-id> --flavor m1.tiny --nic net-id=<tenant-internal-network-id> VM2`

| Confirm that both the VMs have successfully booted up.
|   `nova list`
|   `nova console-log VM1`
|   `nova console-log VM2`

| Add the necessary security group ingress rules.
|   `source openrc demo demo`
| # SSH access to the VMs
|   `neutron security-group-rule-create --direction ingress --protocol tcp --port-range-min 22 --port-range-max 22 --remote-ip-prefix 10.0.0.0/24 default`
| # Permit IPv6 Router Advts from the vRouter internal interface to the VMs.
|   `neutron security-group-rule-create --direction ingress --ethertype IPv6 --protocol icmpv6 --port-range-min 134 --remote-ip-prefix fe80::/64 default`
  
| SSH to the cirros VMs to check the IPv6 forwarding use-case.
|   `sudo ip netns`
|   `sudo ip netns exec <router2-namespace> bash`
|   `ssh cirros@<ip-address-of-the-image>`

|   Note: default password of cirros image would be "cubswin:)"
  
| Verify that Cirros image has an IPv6 address assigned via SLAAC with a prefix of "2001:db8:0:2::/64"
|   `ip address`
| # verify that default route points to the LLA of enp0s8 interface of vRouter.
|   `ip -6 route`

| Try pinging to the internal router interface of router1 (i.e., 2001:db8:0:1::1/64)
|   `ping6 2001:db8:0:1::1/64`

| If all goes well, ping6 should succeed which shows that vRouter is forwarding the IPv6 traffic of instances on the tenant-internal-network.

| At this state, we can create a snapshot of the CentOSvRouter and use it in any other similar OpenStack setup.
|   `nova image-create <CentOSvRouter-id> <Snapshot-name>`
|   `nova image-list #You will find the snapshot you just created above.`

