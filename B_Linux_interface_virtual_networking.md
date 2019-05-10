[Redhat linux interface virtual networking](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/)

- Linux has rich virtual networking capabilities that are used as basis for hosting `VMs` and `containers`, as well as `cloud environments`
- A list of interfaces can be obtained using the command ip link help.

# Bridge:
- A `Linux bridge` behaves like a `network switch`. 
- It `forwards packets` `between interfaces` that `are connected to it`. 
- It’s `usually used` for `forwarding packets on routers`, on `gateways`, or `between VMs` and `network namespaces on a host`. 
- It also `supports STP, VLAN filter, and multicast snooping`.
- Use a bridge `when you want to establish communication channels` `between VMs, containers, and your hosts`.
```
+--------------------------------------------------------------------------+
|   +-----------------+       +-----------------+     +-----------------+  |
|   |     VM1         |       |     VM2         |     |     netns 1     |  |
|   |                 |       |                 |     |                 |  |
|   |   +---------+   |       |   +---------+   |     |   +---------+   |  |
|   |   |  eth0   |   |       |   |  eth0   |   |     |   | veth0   |   |  |
|   +-----------------+       +-----------------+     +-----------------+  |
|       |  tap1   |               |  tap2   |             | veth1   |      |
|       +---------+               +-----+---+             +---------+      |
|            +-----+                    |               +-----+            |
|                  |                    |               |                  |
|             +----+--------------------+---------------+--+               |
|             |                  br0                       |               |
|             +---------------------+----------------------+               |
|                                   |                                      |
|                                   |                                      |
|   HOST               +------------+---------+                            |
|                      |         eth0         |                            |
+----------------------+------------+---------+----------------------------+
                                    |
                                    |
                                    |
                                    |
         +--------------------------+---------------------------+
         |                        SWITCH                        |
         +------------------------------------------------------+
```

- Here’s how to create a bridge:
```
# ip link add br0 type bridge
# ip link set eth0 master br0
# ip link set tap1 master br0
# ip link set tap2 master br0
# ip link set veth1 master br0
```
- This creates a `bridge device named br0` and sets two TAP devices (tap1, tap2), a VETH device (veth1), and a physical device (eth0) as its slaves, as shown in the diagram above.

- bridge interface show:
```
$ ip -d link show cni0
34: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 72:25:67:79:9c:f9 brd ff:ff:ff:ff:ff:ff promiscuity 0 
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.72:25:67:79:9c:f9 designated_root 8000.72:25:67:79:9c:f9 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer   34.10 vlan_default_pvid 1 vlan_stats_enabled 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 4 mcast_hash_max 512 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

```
$ brctl show
bridge name bridge id       STP enabled interfaces
cni0        8000.02075448cd83   no      veth187699d8
                                        veth7bac57b2
                                        vethbc4d7957
docker0     8000.02420b878029   no
```

# VETH:
- The VETH (virtual Ethernet) device is a `local Ethernet tunnel`. 
- Devices are `created in pairs`, as shown in the diagram below.

- `Packets transmitted` on `one device in the pair` are `immediately received` on the `other device`. 
- When either `device is down`, the `link state of the pair is down`.

```
+--------------------------------------------------------------------------------------+
|                                                                                      |
|      +--------------+                                                                |
|      |              |                                                                |
|      |  Netns1      |                                                                |
|      |              |                                                                |
|      |  +--------+  |                                                                |
|      |  |        |  |      VETH (virtual Ethernet) device is a local Ethernet tunnel |
|      |  | Veth0  |  |                                                                |
|      +--+----+---+--+                                                                |
|              |                                                                       |
|              |                                                                       |
|       +------+-+                                                                     |
|       |  Veth1 |                                                                     |
|       +--------+------------------+                                                  |
|       |            br0            |                                                  |
|       +---------+---------+-------+                    HOST                          |
|                 |  eth0   |                                                          |
+-----------------+-----+---+----------------------------------------------------------+
                        |
                        |
                        |
              +---------+----------+
              |       SWITCH       |
              +--------------------+
```

- `Use a VETH configuration` `when namespaces` need to `communicate to the main host namespace` or `between each other`.
- Here’s how to set up a VETH configuration:
```
# ip netns add net1
# ip netns add net2
# ip link add veth1 netns net1 type veth peer name veth2 netns net2
```

- This creates two namespaces, `net1 and net2`, and a `pair of VETH devices`, and it `assigns` `veth1 to namespace net1` and `veth2 to namespace net2`. These `two namespaces are connected with this VETH pair`. Assign a pair of IP addresses, and you can ping and communicate between the two namespaces.

- In k8s: When the pod is spun up, a veth pair is created, one end of the veth pair is eth0 of pod, and the other end [named: vethxxx] is an interface created in root ns. Figuring out the mapping of eth0 ← → vethxxx

```
[dat.hoang@infra16 ~]$ ip -o link 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000\    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000\    link/ether 00:50:56:b6:19:45 brd ff:ff:ff:ff:ff:ff
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default \    link/ether 02:42:3c:b1:25:62 brd ff:ff:ff:ff:ff:ff
33: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default \    link/ether 26:ff:28:64:60:9c brd ff:ff:ff:ff:ff:ff
34: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000\    link/ether 72:25:67:79:9c:f9 brd ff:ff:ff:ff:ff:ff
35: veth9b138b7c@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether a2:b5:82:74:4e:46 brd ff:ff:ff:ff:ff:ff link-netnsid 0
36: vethd1e52dfe@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether 82:ef:c3:7e:2f:ac brd ff:ff:ff:ff:ff:ff link-netnsid 1
51: veth2765d030@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether 02:22:d1:03:87:d3 brd ff:ff:ff:ff:ff:ff link-netnsid 2
52: veth30b982c6@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether 36:f5:19:da:92:be brd ff:ff:ff:ff:ff:ff link-netnsid 3
53: veth484936ff@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether 16:65:75:ce:89:17 brd ff:ff:ff:ff:ff:ff link-netnsid 4
55: veth5b6344e4@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether 12:25:52:66:9f:c9 brd ff:ff:ff:ff:ff:ff link-netnsid 6
56: vethc62596e7@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether ce:93:10:6d:32:3a brd ff:ff:ff:ff:ff:ff link-netnsid 7
57: vethf646d660@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether ba:9c:9d:5c:30:f6 brd ff:ff:ff:ff:ff:ff link-netnsid 8
58: veth7c77db82@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether 9a:3b:7c:07:33:fa brd ff:ff:ff:ff:ff:ff link-netnsid 9
59: veth27874560@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether ae:4a:7e:ba:33:cd brd ff:ff:ff:ff:ff:ff link-netnsid 10
60: vethd1136d6f@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether e6:b7:04:bb:45:f8 brd ff:ff:ff:ff:ff:ff link-netnsid 11
61: vetha4c0fd38@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether b2:25:52:ab:13:ee brd ff:ff:ff:ff:ff:ff link-netnsid 12
62: veth140ad570@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default \    link/ether 92:5b:80:f9:8d:9b brd ff:ff:ff:ff:ff:ff link-netnsid 5
```


# VXLAN:
- VXLAN (Virtual eXtensible Local Area Network) is a `tunneling protocol` designed to `solve the problem of limited VLAN IDs (4,096)` in IEEE 802.1q. It is described by IETF RFC 7348.
- With a `24-bit segment ID`, aka VXLAN Network Identifier (VNI), `VXLAN allows up to 2^24 (16,777,216) virtual LANs`, which is 4,096 times the VLAN capacity.
- VXLAN `encapsulates Layer 2 frames` with a `VXLAN header into a UDP-IP packet`, which looks like this: [packet encapsulate](https://developers.redhat.com/blog/wp-content/uploads/2018/10/vxlan_01.png)

- VXLAN is `typically deployed` in `data centers on virtualized hosts`, which may be `spread across multiple racks`. [vxlan topo](https://developers.redhat.com/blog/wp-content/uploads/2018/10/vxlan.png)

- Here’s how to use VXLAN:
```
# ip link add vx0 type vxlan id 100 local 1.1.1.1 remote 2.2.2.2 dev eth0 dstport 4789
```

```
$ ip -d l show flannel.1
33: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default 
    link/ether 26:ff:28:64:60:9c brd ff:ff:ff:ff:ff:ff promiscuity 0 
    vxlan id 1 local 172.18.1.216 dev ens192 srcport 0 0 dstport 8472 nolearning ageing 300 noudpcsum noudp6zerocsumtx noudp6zerocsumrx addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```