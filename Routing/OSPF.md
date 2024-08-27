Short for "**Open Shortest Path First**", uses the Shortest Path First algorithm of Dutch computer scientist Edsger Dijkstra (**Dijkstra's algorithm**).
- An IGP (Interior Gateway Protocol) that uses the Link State algorithm.
- Three versions:
	- OSPFv1 (1989): OLD, not in use anymore
	- OSPFv2 (1998): Used for IPv4
	- OSPFv3 (2008): Used for IPv6 (Can also be used for IPv4, but usually v2 is used)
## Overview
In the following network topology,
	![[Pasted image 20240827161701.png]]
Following can be seen from R1 using the `show ip protocols` command:
	![[Pasted image 20240827161432.png]]
- `Routing Protocol is "ospf 1"`
	- The **OSPF process ID** is set to 1. This is locally significant, so they don't have to match between OSPF neighbours.
	- A router can run multiple OSPF processes at once, and this ID is used to identify between them.
	- Unrelated to the area!
- **Router ID**: A 32-bit number formatted like a dotted-decimal IP address, but can be changed. 
	- Order of priority:
		1. Manual configuration
		2. Highest IP address on a loopback interface
		3. Highest IP address on a physical interface
	- The loopback interface was manually configured as `1.1.1.1`, so it became the router ID.
- `This is an autonomous system boundary router`
	- This router is set as an **ASBR** that advertises a default route into the OSPF domain, using the command `ip route 0.0.0.0 0.0.0.0 203.0.113.2`. 
## OSPF Process
Three main steps in the process of sharing LSAs and determining the best route to each destination in the network:
1. Become neighbours with other routes connected to the same segment.
2. Exchange LSAs with neighbour routes (through [[#LSA Flooding]])
3. Calculate the best route to each destination, and insert them into the routing table. (This is done independently for each router)
## LSA Flooding
Routers store information about the network in **LSAs (Link State Advertisements)**, which are organised in a structure called the **LSDB (Link State Database)**.
- LSDBs are identical for all routers in the area - they contain LSAs for all of the different links in the network.
- Routers will **flood** LSAs until all routers in the OSPF **area** develop the same map of the network (LSDB). ![[Pasted image 20240827140032.png]]
- Each individual LSA has an aging timer of 30 minutes by default.
	- LSA will be flooded every 30 minutes.
## OSPF Areas
OSPF uses **areas** to divide up the network. 
- Small networks can be single-are without any negative effects, but larger networks may have some problems:
	1. SPF algorithm takes more time to calculate routes.
	2. SPF algorithm takes more processing power on the routes. (exponentially!)
	3. Larger LSDB takes up more memory on the routers.
	4. Any small change in the network causes every router to flood LSAs and run the SPF algorithm again.
	These can be avoided by dividing a large OSPF network into several smaller areas.
### Terminology
![[Screenshot 2024-08-27 141812.png]]
- **Area**: A set of routers and links that share the same LSDB.
- **Backbone Area (Area 0)**: The area that all other areas must connect to.
- **Internal Routers**: Routers with all interfaces in the same area.
- **Area Border Routers (ABR)**: Routers with interfaces in multiple areas.
	- ABRs maintain a separate LSDB for each area they are connected to. Recommended to have max 2 areas connected, since more connection can overburden the router.
- **Backbone Routers:** Routers connected to the backbone area (area 0).
	- Backbone Routers can be ABRs.
- **Intra-area** Route: Route to a destination inside the same OSPF area. ![[Pasted image 20240827142427.png]]
- **Interarea Route**: Route to a destination in a different OSPF area. ![[Pasted image 20240827142354.png]]
- **Autonomous System Boundary Router (ASBR)**: An OSPF router that connects the OSPF network to an external network.
	- E.g., If R1 is connected to the internet, it becomes an ASBR.
### Area Rules
1. OSPF areas should be **contiguous**. (They should be touching along the boundary.)
	- This is not allowed, area on the right should be a separate area:  ![[Pasted image 20240827142553.png]]
2.  All OSPF areas must have **at least one ABR** connected to the backbone area.
	- This is not allowed: ![[Pasted image 20240827142529.png]]
3. OSPF interfaces in the **same subnet** must be in the **same area**.
	- This is not allowed: ![[Pasted image 20240827142650.png]]

## OSPF Cost
OSPF's metric, automatically calculated based on the bandwidth (speed) of the interface.
- Cost = Interface Bandwidth / **Reference Bandwidth**
- Default reference bandwidth is 100 mbps.
	- Reference: 100 mbps / Interface: 10 mbps (Ethernet) = cost of 10
	- Reference: 100 mbps / Interface: 100 mbps (FastEthernet) = cost of **1**
	- Reference: 100 mbps / Interface: 1000 mbps (Gigabit Ethernet) = cost of **1**
	- Reference: 100 mbps / Interface: 10000 mbps (10-Gigabit Ethernet) = cost of **1**
All values less than 1 will be converted to 1, so FastEthernet, Gigabit Ethernet, 10Gig Ethernet, etc. are equal and will have a cost of 1 by default.
	  