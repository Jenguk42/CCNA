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
- Refer to [RIP Configuration](<Router CLI Commands#RIP Configuration>) for configuration commands.
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
Cisco proprietary, but published openly. Still mostly limited to Cisco devices so OSPF is used more commonly. (Refer to [[Router CLI Commands#EIGRP Configuration]] for configuration commands.)
- Considered an 'advanced'/'hybrid' distance vector routing protocol, much faster than RIP in reacting to changes in network.
- No 15 'hop-count' limit
- Messages are ==multicast to `224.0.0.10`==.
- **NOTE: EIGRP is the only IGP that can perform unequal-cost load balancing!**
	- By default, it performs ECMP (Equal Cost Multi-Path) load-balancing over 4 paths like RIP.
	- Can be configured to load-balance over multiple paths that have different cost, or in proportion to the bandwidth.
- Example: ![[Pasted image 20240827114354.png]]
	- `Routing Protocol is "eigrp 100"
		- 100 is the AS number that is configured.
	- Two separate AD values (Distance):
		- Internal routes: 90
		- External routes: 170 (routes from outside EIGRP that are inserted to EIGRP.)
	- `EIGRP maximum metric vairance 1`
		- Variance 1 = Only ECMP load-balancing will be preformed.
		- This can be configured to allow **unequal-cost load-balancing**: Refer to [[Router CLI Commands#Unequal-Cost Load-Balancing]]
### EIGRP Metric
By default, EIGRP uses **bandwidth** and **delay** to calculate metric.
- Formula: (\[K1 \* bandwidth + (K2 \* bandwidth) / (256 - load) + K3 \* delay] \* \[K5 / (reliability + K4)]) * 256*
- Default K values
	- K1 = 1 (Multiplied by interface bandwidth)
	- K2 = 0
	- K3 = 1 (Multiplied by delay)
	- K4 = 0
	- K5 = 0
- Metric = bandwidth + delay 
	- Bandwidth of the **slowest** link in the path + **sum** of delay values of all the link in the path
- Important terminologies
	- E.g., R1 is trying to reach the `192.168.4.0/24` network.	![[Screenshot 2024-08-27 120810.png]]
	- Different distances shown for different routes (Through R2 and through R3) ![[Pasted image 20240827121144.png]]
		- FD through R2: 28672 (RD from R2: 28416)
		- FD through R3: 30976 (RD from R3: 28416)
	- ==**Feasible Distance**== = The router's best metric along a path to a destination 
		- E.g., R1's metric to the destination (R1 to R2, R2 to R4, and R4 to send traffic out of its own interface), shown in red
	- ==**Reported Distance (Advertised Distance)**== = The next hop/neighbour's metric to the route's destination.
		- E.g., R1's neighbour R2's metric to the destination, shown in blue
	- ==**Successor**==: The route with the lowest metric to the destination *(the best route)*
		- E.g., R2 is the successor of R1, since the route via R2 has a lower metric.
		- There can be multiple successors if they have the same metric. EIGRP will do ECMP load balancing.
	- ==**Feasible Successor**==: An alternate route to the destination *(not the best route)* which meets the feasibility condition, guaranteed to be loop-free.
	- **==Feasibility condition==**: A route is considered a feasible successor if its **reported distance** is lower than **successor route's feasible distance**.
		- E.g., R3's reported route is 28416. The successor route via R2's feasible distance is 28672. 28416 is less than 28672, so the route via R3 is a feasible successor. ![[Pasted image 20240827133326.png]]
		- **Loop prevention mechanism** - if a route meets the feasibility requirement, it is guaranteed not to be a looped route.
	- EIGRP will only perform unequal-cost load-balancing over **feasible successor routes.** If a route does not meet the feasibility requirement, it will NEVER be selected for load-balancing, regardless of the variance.
### Unequal-Cost Load-Balancing
Other routing protocols only support ECMP (Equal Cost Multi-Path).
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