NAME
       ip - show / manipulate routing, network devices, interfaces and tunnels

SYNOPSIS
       ip [ OPTIONS ] OBJECT { COMMAND | help }

       ip [ -force ] -batch filename

       OBJECT := { link | address | addrlabel | route | rule | neigh | ntable | tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm | netns | l2tp |
               tcp_metrics | token | macsec }

       OPTIONS := { -V[ersion] | -h[uman-readable] | -s[tatistics] | -d[etails] | -r[esolve] | -iec | -f[amily] { inet | inet6 | ipx | dnet | link } | -4 | -6 |
               -I | -D | -B | -0 | -l[oops] { maximum-addr-flush-attempts } | -o[neline] | -rc[vbuf] [size] | -t[imestamp] | -ts[hort] | -n[etns] name | -a[ll] |
               -c[olor] -br[ief] }

COMMAND
       Specifies the action to perform on the object.  The set of possible actions depends on the object type.  As a rule, it is possible to `add, delete and show`

# COMMON USE:
## OPTIONS:
```
       -c, -color
              Use color output.   

       -s, -stats, -statistics
              Output more information. If the option appears twice or more, the amount of information increases.  As a rule, the information is statistics or some
              time values.

       -d, -details
              Output more detailed information.

       -n, -netns <NETNS>
              switches ip to the specified network namespace NETNS.  Actually it just simplifies executing of:

              ip netns exec NETNS ip [ OPTIONS ] OBJECT { COMMAND | help }

              to

              ip -n[etns] NETNS [ OPTIONS ] OBJECT { COMMAND | help }
```

## OBJECT
```
       address
              - protocol (IP or IPv6) address on a device.

       link   - network device.

       monitor
              - watch for netlink messages.

       neighbour
              - manage ARP or NDISC cache entries.

       netns  - manage network namespaces.

       ntable - manage the neighbor cache's operation.

       route  - routing table entry.

       tunnel - tunnel over IP.

       l2tp   - tunnel ethernet over IP (L2TPv3).
```


- Example:
```
$ ip -c -d link show flannel.1 
33: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default 
    link/ether 26:ff:28:64:60:9c brd ff:ff:ff:ff:ff:ff promiscuity 0 
    vxlan id 1 local 172.18.1.216 dev ens192 srcport 0 0 dstport 8472 nolearning ageing 300 noudpcsum noudp6zerocsumtx noudp6zerocsumrx addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

```
$ ip -d link show cni0
34: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 72:25:67:79:9c:f9 brd ff:ff:ff:ff:ff:ff promiscuity 0 
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.72:25:67:79:9c:f9 designated_root 8000.72:25:67:79:9c:f9 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer   34.10 vlan_default_pvid 1 vlan_stats_enabled 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 4 mcast_hash_max 512 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

```
$ ip -d -c -s neigh show
10.244.2.13 dev cni0 lladdr b6:30:cf:9a:fe:4f used 1274296/1268224/1268203 probes 1 STALE
10.244.2.28 dev cni0 lladdr 82:2f:2f:5d:f5:3b ref 1 used 1266654/0/1266652 probes 1 REACHABLE
10.244.2.3 dev cni0 lladdr d2:30:b4:c7:3f:1e used 82404/82399/82378 probes 1 STALE
10.244.2.18 dev cni0 lladdr 6a:21:c3:d2:7e:fc used 1267685/1267685/1267664 probes 4 STALE
172.18.1.217 dev ens192 lladdr 00:50:56:b6:b3:60 ref 1 used 1267815/2/1267810 probes 1 REACHABLE
```