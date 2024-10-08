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
## OSPF Network Types
Type of connection between OSPF neighbours (Ethernet, etc.)
![[Pasted image 20240829110743.png]]
Three main OSPF network types:
- **Broadcast** (DOMINANT!)
	- Enabled by default on **Ethernet** and FDDI (Fiber Distributed Data Interfaces) interfaces
- **Point-to-Point**
	- Enabled by default on PPP (Point-to-Point Protocol) and HDLC (High-Level Data Link Control) interfaces
- Non-broadcast
	- Enabled by default on Frame Relay and X.25 interfaces
	- Neighbours have to be manually configured.
	- Hello timer of 30 secs, Dead timer of 120 secs
- Network type can be configured using the command `ip ospf network TYPE`. 
	- Not all network types work on all link types!
	- E.g., a serial link cannot use the broadcast network type, because serial links don't support Layer 2 broadcast frames.
### Broadcast Network Type
In the broadcast network type, **routers will only form a full OSPF adjacency with the DR and BDR of the segment.**
![[Pasted image 20240828153033.png]]
- Enabled on **Ethernet** and **FDDI** interfaces by default.
- Routers **dynamically discover** neighbours by sending/listening for OSPF Hello messages using multicast address `224.0.0.5`. 
	- Messages to the DR/BDR are multicast using address `224.0.0.6`.
- A **DR (designated router)** and **BDR (backup designated router)** must be elected on each subnet.
	- DR and BDR will form a FULL adjacency with ALL routers in the subnet.
	- Only DR needs to be elected if there are no OSPF neighbours. (E.g., R1's `G1/0` interface).
- Routers which aren't the DR or BDR become a **DROther**.
	- DROthers form a FULL adjacency only with the DR/BDR.
	- DROthers remain in the 2-way state with other DROthers, not forming full adjacencies.
- LSAs are only shared with DR and BDR (DROthers do not exchange LSAs). 
	- This reduces the amount of LSAs flooding the network, minimising the network traffic.
- Example Network Topology:	![[Pasted image 20240829101945.png]]![[Pasted image 20240829092042.png]]
	- `Nbrs`: `F` indicates the number of full adjacencies, and `C` indicates the total counts of neighbours
		- R3 has 2 full adjacencies with R2 and R4.
		- R3 has a total of 3 neighbours: R2, R4 and R5.
	- `State`: Shows if the interface is DROther, DR, or BDR. 
### Point-to-Point Network Type
![[Pasted image 20240829103634.png]]
- Enabled on [[#Serial Interfaces]] using the **PPP** or **HDLC** encapsulations by default.
- Routers **dynamically discover** neighbours by sending/listening for OSPF Hello messages using multicast address `224.0.0.5`.
- However, a **DR** and **BDR** IS **NOT ELECTED**, since these encapsulations are used for 'point-to-point' connection.
- The two routers will form a Full adjacency with each other.
#### Serial Interfaces
![[Pasted image 20240829105600.png]]
Old technology that is not used widely
- Each side of a serial connection functions as:
	- **DCE** (Data Communications Equipment)
	- **DTE** (Data Terminal Equipment)
- DCE **specifies the clock rate** (speed) of the connection, using `clock rate SPEED_IN_BPS`.
- **Default encapsulation on a serial interface is HDLC**. (or cHDLC, Cisco HDLC)
	- No MAC address field
	- You can manually configure the encapsulation to PPP using the command `encapsulation PPP` on the interface.
- Encapsulations should **match between both ends**, or the interface will go down!
#### DR & BDR Election
DR/BDR election is 'non pre-emptive'. Once the DR/BDR are selected they will keep their role until OSPF is reset, the interface fails/is shut down, etc.
- Order of priority when choosing the DR of the subnet:
	1. Highest OSPF Interface Priority
		- All interfaces have the default priority of 1 by default
		- This can be manually configured by `ip ospf priority PRIORITY`, with a range of 0 to 255
		- If the priority is set to 0, the router cannot be the DR/BDR of the subnet.
	2. Highest OSPF Router ID
- 'First place' becomes the DR, and 'Second place' becomes the BDR
- If the DR goes down, **BDR instantly becomes the new DR** even if it doesn't have the highest priority. Then an election is held for the next BDR.
	- E.g., If R2's priority is configured to be 255 and the OSPF processes are cleared, R4 will be the next DR because it was previously the BDR. Then R2 becomes the BDR. 
## OSPF Process
Three main steps in the process of sharing LSAs and determining the best route to each destination in the network:
1. Become neighbours with other routes connected to the same segment.
	- **Down, Init, 2-way States**
2. Exchange LSAs with neighbour routes (through [[#LSA Flooding]])
	- **Exstart, Exchange, and Loading States**
3. Calculate the best route to each destination, and insert them into the routing table. (This is done independently for each router)
## OSPF Neighbours
Making sure that routers **successfully become OSPF neighbours** is the main task in configuring and troubleshooting OSPF, because the routers will automatically share network information, calculate routes, etc once they become neighbours.

Different types of messages are sent to OSPF neighbours at different states:
![[Pasted image 20240828151924.png]]
### Hello Messages
When OSPF is activated on an interface, the router starts sending OSPF **hello messages** out of the interface at regular intervals, which are used to introduce the router to potential OSPF neighbours.
- This interval is determined by the **hello timer**, 10 sec by default on an Ethernet connection. 
- Hello messages are multicast to `224.0.0.5` (multicast addresses for all OSPF routers).
- OSPF messages are encapsulated in an IP header, with a value of 89 in the Protocol Field.
### Neighbour/ Adjacency Requirements
For two routers to become OSPF neighbours:
1. Interfaces must be in the same **area**.
2. Interfaces must be in the same **subnet**.
3. OSPF process must not be `shutdown`.
4. OSPF **Router IDs** must be unique.
5. **Hello and Dead timers** must match.
6. **Authentication settings** must match.
7. IP **MTU (Maximum Transmission Unit) settings** must match. (Refer to [[IPv4 Header]]).
8. OSPF **Network Type** must match.
	- May seem fine because both routers are in FULL state, but the routing table is not updated correctly.
Even if rules 7 and 8 don't match, the routers will become OSPF neighbours; however, they will not function properly. 
### Neighbouring States
The process of Routers becoming OSPF neighbours is as follows. Assume OSPF is already enabled in R1. ![[Pasted image 20240828101626.png]]
1. **Down State**: R1 doesn't know about any OSPF neighbours yet.
	![[Pasted image 20240828110554.png]]
	1. OSPF is activated on R1's G0/0 interface.
	2. It sends an OSPF Hello message to `224.0.0.5`. (R1 doesn't know about R2, so the neighbour router ID field is `0.0.0.0`.)
2. **Init State**: R2 receives a Hello packet from R1, but R2's RID is not in the Hello packet. 
	![[Pasted image 20240828110817.png]]
	1. R2 adds an entry for R1 to its OSPF neighbour table, where the relationship with R1 is now in the **Init** state.
3. **2-way State**: R1 receives a Hello packet with its own RID in it.
	![[Pasted image 20240828111331.png]]
	1. R2 sends the Hello packet containing the RID of both routers.
	2. R1 adds an entry for R2 to its OSPF neighbour table, where the relationship with R2 is now in the **2-way** state.
	3. R1 sends another Hello message, this time containing R2's RID. R2 updates its relationship with R1.
	4. All conditions have been met for the routers to become OSPF neighbours, and they are now ready to share LSAs to build a common LSDB.
	5. DR (Designated Router) and BDR (Backup Designated Router) will be elected. (Refer to [[#DR & BDR Election]])
4. **Exstart State**: Routers decide which one will start the exchange, to prepare for the exchange state.
	![[Pasted image 20240828111734.png]]
	1. The routers exchange DBD (Database Description) packets.
	2. The router with the higher RID will become the **Master** and initiate the exchange. The router with the lower RID will become the **Slave**.
5. **Exchange State**: The routers exchange DBDs which contain a list of the LSAs in their LSDB, to check which ones they should receive.
	- These DBDs does not include detailed information about the LSAs, they are just basic information telling the neighbour what LSAs they have!
	![[Pasted image 20240828111931.png]]
	1. The routers exchange DBD (Database Description) packets. (Master router, R2 in this case, sends the DBD first.)
	2. The routers compare the information in the DBD they received to the information in their own LSDB, then determine which LSAs they must receive from their neighbour.
6. **Loading State**: LSR (Link State Request) messages are used to request LSU (Link State Update) messages, then LSAck messages are used to acknowledge the receival. 
	- LSR are used to request any missing LSAs to make sure each router has the same LSAs.
	![[Pasted image 20240828112646.png]]
	1. R1 sends LSRs to R2 in order to request any LSAs it doesn't have.
	2. R2 sends the LSAs in LSUs.
	3. R1 sends LSAcks to acknowledge that it received the LSAs.   
7. **Full State**: Routers have a full OSPF adjacency and identical LSDBs. Neighbour adjacency is maintained by sending and listening for Hello packets.
	1. Hello packets are sent every 10 seconds by default.
	2. Every time a Hello packet is received, the 'Dead' timer (40 sec by default) is reset.
	3. If the Dead timer counts down to 0 and no Hello message is received, the neighbour is removed.
	4. If the network changes, the routers continue to automatically share LSAs to make sure each router has a complete and accurate map on the network (LSDB).
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
### Reference Bandwidth
OSPF's metric, automatically calculated based on the bandwidth (speed) of the interface. **Reference bandwidth should be consistent across all OSPF routers** in the network to provide a consistent cost.
- Cost = Interface Bandwidth / **Reference Bandwidth**
- Default reference bandwidth is 100 mbps.
	- Reference: 100 mbps / Interface: 10 mbps (Ethernet) = cost of 10
	- Reference: 100 mbps / Interface: 100 mbps (FastEthernet) = cost of **1**
	- Reference: 100 mbps / Interface: 1000 mbps (Gigabit Ethernet) = cost of **1**
	- Reference: 100 mbps / Interface: 10,000 mbps (10-Gigabit Ethernet) = cost of **1**
All values less than 1 will be converted to 1, so FastEthernet, Gigabit Ethernet, 10Gig Ethernet, etc. are equal and will have a cost of 1 by default.
- Do this by command `auto-cost reference-bandwidth MBITS_PER_SECOND`
- Reference bandwidth should be configured greater than the fastest links in your network (to allow future upgrades).
- E.g., Configured reference bandwidth is 100,000 mbps.
	- Reference: 100,000 mbps / Interface: 100 mbps (FastEthernet) = cost of **1000**
	- Reference: 100,000 mbps / Interface: 1000 mbps (Gigabit Ethernet) = cost of **100**
Refer to [OSPF Configuration](<Router CLI Commands#OSPF Configuration>)
### Cost Calculation
OSPF cost to a destination is the total cost of the 'outgoing/exit' interfaces.
- Loopback interfaces have a cost of 1.
- Example Network Topology: 
	![[Pasted image 20240828100026.png]]
	- R1's cost to reach `192.168.4.0/24` is: 100 (R1 `G0/0`) + 100 (R2 `G1/0`) + 100 (R4 `G1/0`) = 300
	- R1's cost to reach `2.2.2.2`, R2's loopback interface is: 100 (R1 `G0/0`) + 1 (R2 `L0`) = 101
## OSPF LSA Types
The OSPF LSDB is made up of LSAs. There are 11 types, and three important:
![[Pasted image 20240829152249.png]]
- **Type 1 (Router LSA)**
	- Every OSPF router generates this type of LSA.
	- It identifies the router using its router ID.
	- It also lists networks attached to the router's OSPF-activated interfaces.
- **Type 2 (Network LSA)**
	- Generated by the DR of each 'multi-access' network (i.e. Ethernet network, using the **broadcast** network type).
	- Lists the routers which are attached to the multi-access network.
- **Type 5 (AS-External LSA)**
	- Generated by ASBRs to describe routes to destinations outside of the AS (OSPF domain).