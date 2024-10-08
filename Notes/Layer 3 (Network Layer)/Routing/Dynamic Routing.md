Configure a dynamic routing protocol on the router, and let the router take care of finding the bets routs to destination networks.
- E.g., Routers will automatically inform each other how to reach a new destination network.
## Example Topology
![[Pasted image 20240822145634.png]]
Routes that are automatically added without configuration:
- Network route: A route to a network/subnet (mask length < 32)
	- `10.0.12.0/30`, `10.0.13.0/30`
- Host route: A route to a specific host (/32 mask)
	- Specific addresses configured to R1's `G0/0` and `G0/1` interfaces
	- `10.0.12.1/32`, `10.0.13.1/32`
## Dynamic Routing Protocol
Purpose: To let the router select the best route to the destination.
Routers can use dynamic routing protocols to advertise information about their connected routes and routes they have learned from other routes.
- Routers form adjacencies/ neighbour relationships/ neighbourships with adjacent routes to exchange this information.
- If there is a connection error, routers will **adjust the routing table** and remove the invalid routes.
	- Prevents the router from continuously sending traffic to a dead-end.
	- If there is a back-up route, the routing table will be adjusted accordingly. 
### Categories: IGP and EGP
![[Pasted image 20240822153341.png]]
- **IGP (Interior Gateway Protocol)**: Used to share routes within a single *autonomous* system (AS), which is a single organisation (E.g., a company)
- **EGP (Exterior Gateway Protocol)**: Used to share routes *between* different autonomous systems.
- Can be broken down to algorithm type (process used to share route information and choose the best route to the destination) ![[Pasted image 20240822153640.png]]
### Network Command
The map is shared by the `network` command (Refer to [[Router CLI Commands]])
- `network IP_ADDRESS`: "Activate IGP on interfaces with an IP address that falls under the given IP range."
![[Pasted image 20240823134517.png]]
- Used in RIP, OSPF, and EIGRP
- Automatically converts the IP address to classful networks.
	- E.g., command `network 10.0.12.0` will be converted to `network 10.0.0.0/8` (Class A network).
	- No need to enter the network mask
- Tells the router to:
	- Look for interfaces with an IP address that is in the specified range.
	- Activate RIP on the interfaces that fall in the range.
	- Form adjacencies with connected RIP neighbours.
	- Advertise **the network prefix of the interface** (NOT the prefix in the `network` command)
- This command doesn't tell the router which networks to advertise, it tells the router **which interfaces to activate RIP on.**
	- The router will advertise the network prefix of these interfaces.
## Dynamic Routing Algorithms
### Distance Vector Algorithm
![[Pasted image 20240822150240.png]]
Works by sharing parts of the routing table with its neighbours.
- Protocols following the algorithm
	- Widely used protocols: Routing Information Protocol (RIP), Enhanced Interior Gateway Routing Protocol (EIGRP)
	- Early examples: RIPv1, Cisco's proprietary protocol IGRP
- Routers operate by the **"routing by rumour" method**
	- Send the following to the directly connected neighbours:
		- Their known destination networks
		- Their metric to reach their known destination networks
	- Routers **don't know about the network beyond its neighbours**, only knows the information that its neighbours tell it.
- Routers only learn the 'distance' (metric) and 'vector' (direction, the next hop router) of each route.
### Link State Algorithm
Works by getting a whole picture of the network.
- Protocols following the algorithm
	- Open Shortest Path First (OSPF), Intermediate System to Intermediate System (IS-IS) 
- Routers create a **'connectivity map'** of the network, then calculate the best routes independently.
	- Each router advertises info about its interfaces (connected networks) to its neighbours.
	- These advertisements are passed along to other routers, until **all routers in the network develop the same complete map.**
- Uses more resources (CPU) on the router, because more information is shared.
- However, tends to be faster in reacting to changes in the network than distance vector protocols.
## Dynamic Routing Protocol Metrics
Used to compare routes learned from the **same** routing protocol.
![[Pasted image 20240822160240.png]]
If multiple routes to a destination is learned, the router determines which route is superior and adds it to the routing table.
- It uses the 'metric' of the route to decide which is superior (lower metric = superior).
- Each routing protocol uses a different metric to determine which route is the best.
## ECMP (Equal Cost Multi-Path) Load Balancing
If a router learns two (or more) routes: 
- Via the same routing protocol,
- To the same destination (same network address, same subnet mask),
- With the same metric,
both will be added to the routing table and traffic will be load balanced over both routes.
This can be done in static routing as well. (Metric of the routes will be 0)
## Administrative Distance
Used to determine which routing protocol is preferred. 
- In rare cases, there may be two IGP (Interior Gateway Protocol) used together (E.g., if two companies connect their networks to share information).
- Different routing protocols use different metrics, so we can use AD to determine which one is preferred.
- AD should be used before comparing metrics to select the best route.
A **lower AD is preferred**, and indicates the routing protocol is considered more 'trustworthy' (more likely to select good routes).
- E.g., RIP only takes hop count into consideration, not bandwidth. So it has a higher AD than OSPF.
### Floating Static Route
AD of a static route can be configured to change the preferred routing protocol. (Refer to [[CLI Commands/Router CLI Commands#Dynamic Route Configuration|Router CLI Commands]])
- This can make static routes less preferred than routes learned by a dynamic routing protocol to the same destination.
	- Make sure the static route's AD is higher than the dynamic route's AD!
- This makes the route inactive unless the route learned by the dynamic routing protocol is removed.
	- E.g., If the remote router stops advertising it, an interface failure causes an adjacency with a neighbour to be lost, etc.
### Cisco Ranks of Administrative Distance: 
Remember AD is considered only to determine which route is placed in the routing table when multiple routes to a destinations are known. 
![[Pasted image 20240822161133.png]]
"If the administrative distance is 255, the router does not believe the source of that route and does not install the route in the routing table."
#### Routing Table Demonstration
![[Pasted image 20240823113116.png]]
- Connected and local routes have AD of 0, but is not displayed in the route table
- Static routes have an AD of 1
![[Pasted image 20240822161647.png]]
- OSPF has an AD of 110 
- The numbers on the right (0, 2, 3) show the metrics
## Passive Interface
Routers will continuously send advertisements out of all their interfaces even if there is no same-protocol enabled neighbours connected. This is unnecessary traffic, so these interfaces should be configured as **passive interface**. 
- Used in RIP, EIGRP, and OSPF.
- Example scenario with RIP: ![[Pasted image 20240823141421.png]]
- Should be used on interfaces which don't have any same-protocol neighbours. 
- Tells the router to stop sending advertisements out of the specified interface.
- The router will still advertise the network prefix of the interface to its RIP neighbours.
- Configuration: [[Router CLI Commands#Passive Interface Configuration]]
## Default Route
Share the default route with the neighbouring same-protocol-enabled routers.
![[Pasted image 20240823143041.png]]
- R1 configured the default route with the command: `ip route 0.0.0.0 0.0.0.0 203.0.113.2`.
- After sharing the default route with `default-information originate`, R2, R3, and R4 learns the route too.
	![[Pasted image 20240823143230.png]]
	- This is an example of RIP enabled routers. Both routes (To R3, `10.0.34.1` via interface `F2/0`; To R2, `10.0.24.1` via interface `G0/0`) are stated since they have the same hop count. 
	- R4 will load-balance traffic over the two routes.
## Loopback Interface
A virtual interface in the router
- Status of the loopback interface is not dependent on a physical interface - not prone to network failure. 
	- Always up/up, unless manually disabled with the `shutdown` command
	- Provides a **consistent IP address** that can be used to **reach/identify the router**, that will never fail
- Recommended to make them as passive interfaces. (Waste of resources to send IGP messages, because they are not connected to any other device.)