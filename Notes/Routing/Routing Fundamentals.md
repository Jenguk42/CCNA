Process that routers use to determine the path that IP packets should take over a network to reach their destination
### Key terms
* **Routing table**: Routers store routes to all of their known destinations to a routing table.
	* When routers receive packets, they look in the routing table to find the best route to forward that packet.
* **Next-hop**: The next router in the path to the destination
* **WAN** (Wide Area Network): A network that extends over a large geographical area
* **Match**: A route matches a packet's destination if the packet's destination IP address is part of the network specified in the route.
	* 192.168.1.2 is a part of the 192.168.1.0/24 network. Therefore the route matches the destination IP.
	* 192.1682.1 is NOT a part of the 192.168.1.0/24 network. Therefore the route does not match the destination IP.
	* If the router does not have a route to the packet's destination, it will **drop** the packet 
		* Different from switches which flood frames if they don't have a MAC table entry for the destination.

A **route** (within the routing table) tells the router:
* To send a packet to destination X, you should send the packet to **next-hop** Y. 
	* Or, if the destination is directly connected to the router (**Connected route**), send the packet directly to the destination. 
	* Or, if the destination is the router's own IP address (**Local route**), receive the packet for yourself (don't forward it). 
### Routing Methods
Two main routing methods (methods that routers use to learn routes) that enable routers to send packets to remote destinations
- **Dynamic Routing**
	- Routers use dynamic routing protocols (e.g., OSPF) to share routing info with each other automatically and build their routing tables
- **[[Static Routing]]**
	- A network engineer/admin manually configures routes on the router
### Connected and Local Routes
These two types of routes are **automatically added** to the routing table when the IP addresses are configured on the router's interfaces.
![[Pasted image 20240814142053.png]]
- Only works for direct connection.
- **Connected route** is a route to the network connected to the interface.
	- Code C in the routing table
	- R1 G0/2 IP = 192.168.1.==1==/24
	- Network Address = 192.168.1.==0==/24 
	- Provides a route to all hosts in that network
		- E.g., 192.168.1.10, 192.168.1.100, 192.168.1.232, etc.
	- /24 means the first three octets are fixed, and the last octet is free. ![[Pasted image 20240814142805.png]]
		- *R1 knows: "If I need to send a packet to any host in the 192.168.1.0/24 network (= within the 192.168.1.1 ~ 192.168.1.255 range), I should send it out of G0/2."*
			- E.g., ==192.168.1.==2, ==192.168.1.==7, ==192.168.1.==89 are **matching**
			- E.g., ==192.168.2==.1 is **not matching**; the router will send the packet using a different route, or drop the packet if there is no matching route
- **Local route** is a route to the exact IP address configured on the interface.
	- Code L in the routing table
	- A /32 netmask is used to specify the exact IP address of the interface.
		- Although R1's G0/2 is configured as 192.168.1.1==/24==, but the connected route is to 192.168.1.1==/32==.
	- /32 means that all 32 bits are 'fixed', and cannot change
		![[Pasted image 20240814150830.png]]
		- *R1 knows: "If I receive a packet destined for this IP address, the message is for me."*
### Route Selection
The router will always choose the most specific matching route = the matching route with the longest prefix length.
![[Pasted image 20240814151629.png]]
* A packet destined for 192.168.1.1 is matched by both routes:
	* 192.168.1.0/24 (Contains 256 different IP addresses, 192.168.1.0 - 192.168.1.255)
	* 192.168.1.1/32 (Contains only 1 IP address, 192.168.1.1)
* R1 receives the packet for itself, rather than forwarding out to G0/2.
	* Local route = keep the packet, don't forward  
* Different than switches, which look for an exact match in the MAC address table to forward frames.