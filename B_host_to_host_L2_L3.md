[host-to-host through-a-router](https://www.practicalnetworking.net/series/packet-traveling/host-to-host-through-a-router/)
[host-to-host through-a-switch](https://www.practicalnetworking.net/series/packet-traveling/host-to-host-through-a-switch/)
[host-to-host Communication](https://www.practicalnetworking.net/series/packet-traveling/host-to-host/)

- through switch: L2 ARP, packet switching

- through router:  -> Encap L3: need to know `IP dest` for encap `IP source + IP dest` in `L3 Header` 
                   -> Encap L2: L2 ARP as for MAC source + M
                   -> Data | L3 Header | L2 Header |  