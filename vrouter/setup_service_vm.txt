================================================
Set up a Service VM Running as a vRouter (SLAAC)
================================================

| # Current network setup for IPv6 router VM on local virtualbox setup
| # /etc/sysconfig/network-scripts/ifcfg-enp0s3
| # Network interface enp0s3 is IPv4 for public internet access
| TYPE="Ethernet"
| BOOTPROTO="dhcp"
| DEFROUTE="yes"
| PEERDNS="yes"
| PEERROUTES="yes"
| IPV4_FAILURE_FATAL="no"
| IPV6INIT="yes"
| IPV6_AUTOCONF="yes"
| IPV6_DEFROUTE="yes"
| IPV6_PEERDNS="yes"
| IPV6_PEERROUTES="yes"
| IPV6_FAILURE_FATAL="no"
| NAME="enp0s3"
| UUID="32bad876-680a-4f78-a364-726eae21bfcf"
| DEVICE="enp0s3"
| ONBOOT="yes"

| # /etc/sysconfig/network-scripts/ifcfg-enp0s8
| # Network interface enp0s8 is IPv6 internal interface to provide IPv6 to internal hosts
| BOOTPROTO=static
| IPV6INIT=yes
| IPV6ADDR="2001:db8:0:2::1/64"
| NAME=enp0s8
| UUID=e931a806-2f76-425d-b035-d37813b81df5
| DEVICE=enp0s8
| ONBOOT=yes
| NM_CONTROLLED=no

| # Disable NetworkManager
| systemctl disable NetworkManager

| # Install dhcp.x86_64, dhcp-common.x86_64, radvd.x86_64 if not already installed
| yum install dhcp-common
| yum install dhcp
| yum install radvd

| # /etc/sysctl.conf Set sysctl to enable IPv6 forwarding
| net.ipv6.conf.all.forwarding=1
| net.ipv6.conf.enp0s3.accept_ra=2
| net.ipv6.conf.enp0s3.accept_ra_defrtr=1
| net.ipv6.conf.enp0s3.router_solicitations=1

| # /etc/radvd.conf
| interface enp0s8
| {
| # This is the primary "on switch" for RADVD
|     AdvSendAdvert on;     
| #
| # These settings determine how often advertisements will be sent every X-Y.
| # X and Y are in seconds.
| # With these settings you will be sending a advert every 60 seconds
| #
|     MinRtrAdvInterval 60;
|     MaxRtrAdvInterval 180;
| #
| # Disable Mobile IPv6 support
| #
|     AdvHomeAgentFlag off;
| #
| # Here we set our managed flags
| #
|     AdvManagedFlag on;
|     AdvOtherConfigFlag on;
| #
| # Enter our IPv6 prefix and CIDR
| #
|     prefix 2001:db8:0:2::/64
|     {
|         AdvOnLink on;
| # On link tells the host that the default router is on the same "link" as it is
|         AdvAutonomous on;
|         AdvRouterAddr off;
|     };
| };

# Enable radvd service
systemctl enable radvd

# In /etc/sysconfig/network add
IPV6FORWARDING=yes

=================================================================
Set up a Service VM Running as a vRouter (DHCPv6 Stateful Server)
=================================================================

| # Current network setup for IPv6 router VM on local virtualbox setup
| # /etc/sysconfig/network-scripts/ifcfg-enp0s3
| # Network interface enp0s3 is IPv4 for public internet access
| TYPE="Ethernet"
| BOOTPROTO="dhcp"
| DEFROUTE="yes"
| PEERDNS="yes"
| PEERROUTES="yes"
| IPV4_FAILURE_FATAL="no"
| IPV6INIT="yes"
| IPV6_AUTOCONF="yes"
| IPV6_DEFROUTE="yes"
| IPV6_PEERDNS="yes"
| IPV6_PEERROUTES="yes"
| IPV6_FAILURE_FATAL="no"
| NAME="enp0s3"
| UUID="32bad876-680a-4f78-a364-726eae21bfcf"
| DEVICE="enp0s3"
| ONBOOT="yes"

| # /etc/sysconfig/network-scripts/ifcfg-enp0s8
| # Network interface enp0s8 is IPv6 internal interface to provide IPv6 to internal hosts
| BOOTPROTO=static
| IPV6INIT=yes
| IPV6ADDR="2001:db8:0:2::1/64"
| NAME=enp0s8
| UUID=e931a806-2f76-425d-b035-d37813b81df5
| DEVICE=enp0s8
| ONBOOT=yes
| NM_CONTROLLED=no

| # Disable NetworkManager
| systemctl disable NetworkManager

| # Install dhcp.x86_64, dhcp-common.x86_64, radvd.x86_64 if not already installed
| yum install dhcp-common
| yum install dhcp
| yum install radvd

| # /etc/sysctl.conf Set sysctl to enable IPv6 forwarding
| net.ipv6.conf.all.forwarding=1
| net.ipv6.conf.enp0s3.accept_ra=2
| net.ipv6.conf.enp0s3.accept_ra_defrtr=1
| net.ipv6.conf.enp0s3.router_solicitations=1

| # /etc/dhcp/dhcpd6.conf
| # DHCP for IPv6 Server Configuration file.

| # Enable RFC 5007 support (same than for DHCPv4)
    allow leasequery;

| # IPv6 address valid lifetime
| #  (at the end the address is no longer usable by the client)
| #  (set to 30 days, the usual IPv6 default)
|     default-lease-time 2592000;

| # IPv6 address preferred lifetime
| #  (at the end the address is deprecated, i.e., the client should use
| #   other addresses for new connections)
| #  (set to 7 days, the  usual IPv6 default)
|     preferred-lifetime 604800;

| # T1, the delay before Renew
| #  (default is 1/2 preferred lifetime)
| #  (set to 1 hour)
|     option dhcp-renewal-time 3600;

| # T2, the delay before Rebind (if Renews failed)
| #  (default is 3/4 preferred lifetime)
| #  (set to 2 hours)
|     option dhcp-rebinding-time 7200;

| # The path of the lease file
|     dhcpv6-lease-file-name "/var/lib/dhcpd/dhcpd6.leases";

| # Set preference to 255 (maximum) in order to avoid waiting for
| # additional servers when there is only one
|     option dhcp6.preference 255;

| # Server side command to enable rapid-commit (2 packet exchange)
|     option dhcp6.rapid-commit;

| # The delay before information-request refresh
| #  (minimum is 10 minutes, maximum one day, default is to not refresh)
| #  (set to 6 hours)
    option dhcp6.info-refresh-time 21600;

| # Set this to `interim` when doing ddns updates
|     ddns-update-style interim;
| 
|     subnet6 2001:db8:0:2::/64 {
|         option dhcp6.name-servers 2001:db8:0:2::1;
|         option dhcp6.domain-search "opnfv.local";
|         ddns-hostname = concat(binary-to-ascii(10, 8, "-", leased-address), ".wired");
|         ddns-domainname = "opnfv.local";
| # Our address range 1000 through 1fff
|         range6 2001:db8:0:2::1000 2001:db8:0:2::1fff;
|     }
| 
| # In /etc/sysconfig/network add
| IPV6FORWARDING=yes

For reference, refer to `How to set up RADVd DHCPv6 and DNS on CentOS 6`_.

.. _`How to set up RADVd DHCPv6 and DNS on CentOS 6`: http://www.percula.info/archives/196

