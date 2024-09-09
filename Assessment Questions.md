## EIGRP Debugging
Refer to: [[Dynamic Routing]], [[RIP & EIGRP]]
### 1. EIGRP Packets
```
Router A# debug eigrp packets 
...
01:39:13: EIGRP: Received HELLO on Serial0/0 nbr 10.1.2.2 
01:39:13: AS 100, 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/0 
01:39:13:      K-value mismatch
```
Router A has received a HELLO packet from a neighbour but is encountering an issue due to a K-value mismatch.
- The output from the **`debug eigrp packets`** command provides detailed information about EIGRP packet exchanges.
- In this case, it shows that Router A has received a HELLO packet from a neighbour but is encountering an issue due to a K-value mismatch.
--- 
1. **`01:39:13: EIGRP: Received HELLO on Serial0/0 nbr 10.1.2.2`**
    - At **01:39:13**, Router A received an **EIGRP HELLO packet** on the **Serial0/0** interface from a neighbour with the IP address **10.1.2.2**.
    - HELLO packets are used in EIGRP to discover and maintain neighbour relationships between routers.
2. **`01:39:13: AS 100, 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/0`**
    - **AS 100**: This refers to the EIGRP **Autonomous System number** that Router A is operating in, which is **100**.
    - The rest of this line (`0x0, Seq 0/0 ...`) represents internal EIGRP debugging information, including sequence numbers, queues, and reliability counters. This is primarily useful for Cisco engineers or deep diagnostics, but it is less relevant for common troubleshooting.
3. **`01:39:13: K-value mismatch`**
    - This message indicates that Router A has detected a **K-value mismatch** with the neighbour at IP address **10.1.2.2**.
    - **K-values** are part of EIGRP's metric calculation formula (configurable parameters). EIGRP uses five K-values (K1 through K5) to define how certain metrics influence route selection:
		- **K1 (Bandwidth)**
		- **K2 (Load)**
		- **K3 (Delay)**
		- **K4 (Reliability)**
		- **K5 (MTU size, typically not used in modern configurations)**
    - Routers must use the same K-values to establish an EIGRP neighbour relationship. 
	    - A mismatch will prevent EIGRP from establishing or maintaining an adjacency with the neighbour.
	    - Therefore router A and its neighbour won't exchange routing information properly. 
### 2. Checking Routes
```
R1#show ip route 
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP 
	D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
	N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
	E1 - OSPF external type 1, E2 - OSPF external type 2 
	i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2 
	ia - IS-IS inter area, * - candidate default, U - per-user static route 
	o - ODR, P - periodic downloaded static route 

Gateway of last resort is 212.50.185.126 to network 0.0.0.0

D   212.50.167.0/24 [90/160000] via 212.50.185.82, 00:05:55, Ethernet1/0
	212.50.166.0/24 is variably subnetted, 4 subnets, 2 masks
D     212.50.166.0/24 is a summary, 00:05:55, Null0
C     212.50.166.1/32 is directly connected, Loopback1
C     212.50.166.2/32 is directly connected, Loopback2
C     212.50.166.20/32 is directly connected, Loopback20
	212.50.185.0/27 is subnetted, 3 subnets 
C     212.50.185.64 is directly connected, Ethernet1/0 
C     212.50.185.96 is directly connected, Ethernet0/0 
C     212.50.185.32 is directly connected, Ethernet2/0 
D*EX 0.0.0.0/0 [170/2174976] via 212.50.185.126, 00:05:55, Ethernet0/0
			   [170/2174976] via 212.50.185.125, 00:05:55, Ethernet0/0
```
Focus on the last two lines:
- `D` indicates that the route was learned via **EIGRP**.
- `*` denotes that this is a **candidate default route** or **gateway of last resort**.
	- This route will be used for packets destined for networks not explicitly listed in the routing table.
- `EX` means the route is an **external EIGRP route**.
	- The route was learned from a source outside the EIGRP Autonomous System (AS), likely redistributed from another routing protocol (e.g., BGP, static route, etc.).
- `0.0.0.0/0` is the **default route**.
	- It matches all IP destinations not covered by a more specific route in the routing table, and directs traffic to any destination for which no more specific route exists.
- `170/2174976` are the **EIGRP administrative distance** and **EIGRP composite metric**, respectively.
	- `170`: The **administrative distance** for external EIGRP routes, which defines the trustworthiness of this route. Since it's an external route, the distance is 170 (internal EIGRP routes have a lower AD of 90).
	- `2174976`: The **EIGRP metric**, which is calculated based on bandwidth, delay, load, and reliability. This value helps determine the best path to the destination (lower is better).
- `via 212.50.185.126` shows the **next-hop IP address**.
	- Traffic destined for `0.0.0.0/0` will be forwarded to the router at IP address `212.50.185.126`.
- `00:05:55` is the amount of time (in `hours:minutes`) since the route was last updated or learned by the router.
- `Ethernet0/0` indicates the **outgoing interface** on the local router that will be used to reach the next hop. 
---
What does this imply?
- **Multiple Next Hops (Load Balancing)**:
    - The last two lines indicate that **load balancing** is occurring over two equal-cost paths. The router can forward traffic destined for unknown networks (`0.0.0.0/0`) to either `212.50.185.126` or `212.50.185.125`, both via the same interface **Ethernet0/0**.
- **Equal-Cost Load Balancing**:
    - Since both paths have the same **EIGRP metric** (2174976) and are both learned from **EIGRP external sources** (EX), the router will balance traffic between these two next hops using a method like per-destination or per-packet load balancing, depending on how the router's forwarding mechanism (e.g., CEF) is configured.
---
How would you confirm load balancing on R1?
- The `traceroute` command helps trace the path that packets take to a destination.
- In the context of load balancing, if you trace to an address that is **not explicitly in the routing table** (i.e., a destination that will match the default route), the router will use the default route (`0.0.0.0/0`) to forward the packet.
- When **load balancing** is occurring, the router will forward packets to different next hops in a round-robin fashion (for equal-cost load balancing) or based on another load-balancing method (such as per-destination or per-packet).
- By running multiple traceroutes to a destination, you can observe if different paths are used (indicating that load balancing is happening).
- Since the destination is not explicitly in the routing table, it forces the router to rely on the default route, which is where load balancing could occur if multiple paths exist.
---
Other Options
- Use an extended ping along with repeated `show ip route` commands to confirm the gateway of last resort address toggles back and forth.
	- This method involves sending multiple pings and observing the routing table with `show ip route` to see if the **gateway of last resort** (default route) is toggling between different next hops.
	- This method does not necessarily reflect load balancing in real-time. The gateway of last resort will not toggle unless there is a failure. Load balancing does not cause the default route to toggle in the routing table.
- Load balancing does not occur over default networks; the second route will only be used for failover.
	- This statement is incorrect. **Load balancing can occur over default networks** as long as there are multiple equal-cost default routes (multiple next-hop addresses for the `0.0.0.0/0` route). If the router has multiple default routes with the same cost, it will perform load balancing.
	- The second route will not be solely used for failover unless unequal-cost paths are involved or configured with specific routing protocols like EIGRP with unequal-cost load balancing (using `variance`).
- Use ping and the `show ip route` command to confirm the timers for each default network reset to 0.
	- These are not reliable indicators of load balancing. Ping just sends packets, and the `show ip route` command displays the routing table, but it won’t show which next hop is being used for each individual ping.
	- Load balancing does not reset any timers in the routing table entries. The **timers** are associated with the uptime of the route, and they do not get reset by load balancing activity.