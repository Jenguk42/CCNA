Not automatically added to the routing table like the connected and local routes (refer to [[Routing Fundamentals]]) - need to be manually configured

![[Pasted image 20240814153912.png]]
PC1 wants to send a packet to PC4.
* It requires **two-way reachability**, so both PCs can send packets to each other
	* Each router in the path needs a route to `192.168.1.0/24` and a route to `192.168.4.0/24`
* Two possible routes:
	* 1. PC1 -> R1 -> R3 -> R4 -> PC4
	* 2. PC1 -> R1 -> R2 -> R4 -> PC4
* Choose via load balancing, main & backup path techniques (Refer to [[Add Later]])

How can we configure R4 so that it knows how to send traffic to `192.168.1.10` (PC1)?
- The connected & local network that is automatically learned by configuring the IP addresses on its interface:
	- G0/0 - C: `192.168.24.0/24`, L: `192.168.24.4/32`
	- G0/1 - C: `192.168.34.0/24`, L: `192.168.34.4/32`
	- G0/2 - C: `192.168.4.0/24`, L: `192.168.4.4/32`
- However, it needs a configuration for other destinations. 
### Default Gateway
* Gateway is an old term for router, so also called default route
* End hosts have to send packets to their **default gateway** to send packets to destinations outside of their local network.
	![[Pasted image 20240815112317.png]]
	* PC1 has to send packets to R1 (`192.168.1.1`) to send packets to destinations outside its local network
	
* Default gateway configuration is also called a default route
	* Route to **`0.0.0.0/0`** (= all netmask bits set to 0, none of the bits in the addresses are fixed)
	* Route that includes everything (all address from `0.0.0.0` to `255.255.255.255`) 
		* The **least specific** route possible, because it includes all possible IP addresses 
		* Opposite of the /32 route (local route) which specifies only one IP address
- If the router does not have any more specific routes that match a packet's destination IP address, it will forward the packet using the default route.
	- "If there aren't any more specific matches for this packet, don't drop it, send it via this route itself."
	
* Often used to direct traffic to the internet.
	![[Pasted image 20240815134222.png]]
	* Destinations in the internal corporate network: more specific routes used  
	* Traffic outside the internal network sent to the internet 
* Configuration refer to [[Router CLI Commands#Routing]]
### Static Route Configuration
- Routers don't need routes to all networks in the path to the destination, it just needs to know the **next hop**.
	![[Pasted image 20240815115402.png]]
	- Blue: connected route, red: static routes (manually configured)
	- These should be manually configured (Refer to [[Router CLI Commands#Static Route Configuration]])
- Code within the routing table is `S` for static
	![[Pasted image 20240815131213.png]]
	- The `[1/0]` means: \[Administrative Distance/Metric] (Refer to [[Add Later]])