## RIP (Routing Information Protocol)
**Distance vector** IGP, which uses **routing-by-rumour logic** to learn/share routes
- Uses hop count as metric, One router = one hop (Bandwidth is irrelevant!)
- **Up to 4 paths** to the same destination will be saved into the routing table, but can be configured otherwise.
- **Maximum hop count is 15** (anything more than that is unreachable)
	- Not used in real networks, can be used in lab environments since it's quick and easy to set
- Three versions
	- IPv4: RIPv1, **RIPv2**
	- IPv6: RIPng (RIP Next Generation)
- Two message types
	- **Request**: To ask RIP-enabled neighbour routers to send their routing table
	- **Response**: To send the local router's routing table to neighbouring routers
	By default, RIP-enabled routers will share their routing table every 30 secs.
- Refer to [[Router CLI Commands#RIP Configuration]] for configuration commands.
### RIPv1
Only advertises *classful* addresses (Class A, B, C) and does not support VLSM, CIDR
- Subnet mask information is not included in response messages
	- E.g., `10.1.1.0/24` will become `10.0.0.0` (Class A address, `/8` assumed) `172.16.192.0/18` will become `172.16.0.0` (Class B address, `/16` assumed) `192.168.1.4/30` will become `192.168.1.0` (Class C address, `/24` assumed) 
- Messages are ==broadcast to `255.255.255.255`==, so all routers on the local segment will receive the messages.
### RIPv2
Supports VLSM, CIDR and includes subnet mask information in advertisements (response messages).
- Messages are ==**multicast** to `224.0.0.9`==.
	- Multicast messages are delivered only to devices that have joined that specific *multicast group*.
## EIGRP (Enhanced Interior Gateway Routing Protocol)
Cisco proprietary, but published openly. Still mostly limited to Cisco devices so OSPF is used more commonly.
- Considered an 'advanced'/'hybrid' distance vector routing protocol, much faster than RIP in reacting to changes in network.
- No 15 'hop-count' limit
- Messages are ==multicast to `224.0.0.10`==.
- The only IGP that can perform **unequal**-cost load balancing.
	- By default, it performs ECMP (Equal Cost Multi-Path) load-balancing over 4 paths like RIP.
	- Can be configured to load-balance over multiple paths that have different cost, or in proportion to the bandwidth.
- Refer to [[Router CLI Commands#EIGRP Configuration]] for configuration commands.

![[Pasted image 20240823152249.png]]
- `Routing Protocol is "eigrp 1"
	- 1 is the AS number that is configured.
- Metric weight: K1 and K3 by default
	- `K1`: Interface bandwidth
	- `K3`: Delay
	- Metric is calculated by bandwidth of the slowest link in the path + sum of delay values of all the link in the path
- Two separate AD values (Distance):
	- Internal routes: 90
	- External routes: 170 (routes from outside EIGRP that are inserted to EIGRP.)
### Router ID
A 32-bit number formatted like a dotted-decimal IP address, but can be changed. 
- Order of priority:
	1. Manual configuration
	2. Highest IP address on a loopback interface
	3. Highest IP address on a physical interface
### Wildcard Masks
![[Pasted image 20240823150457.png]]Inverted subnet mask, which specifies which interface or interfaces to activate EIGRP on: 
- All 1s in the subnet masks are 0 in the equivalent wildcard mask.
- All 0s in the subnet masks are 1 in the equivalent wildcard mask.
- Subtract each octet of the subnet mask from 255. (E.g., 255 - 252 = 3)
	- Subnet mask `/32` = wildcard mask `0.0.0.0`
EIGRP Uses a 'wildcard mask' instead of a regular subnet mask for the `network` command.
- '0' in the wildcard mask = must match
- '1' in the wildcard mask = don't have to match
- E.g., R1 `G2/0` IP address is `172.16.1.14` and the EIGRP `network` command is `172.16.1.0` with the wildcard mask of `0.0.0.15`.
	- `172.16.1.14` = `1010 1100 . 0001 0000 . 0000 0001 . 0000 1110`
	- `172.16.1.0`  = `1010 1100 . 0001 0000 . 0000 0001 . 0000 0000`
	- `0.0.0.15` = `0000 0000 . 0000 0000 . 0000 0000 . 0000 1111`
	- The first 28 bits must match, so EIGRP will be activated on interface `G2/0`.