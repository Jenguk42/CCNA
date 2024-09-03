Works the same as IPv4 routing. However, the two processes and routing tables are separate on the router.
- IPv4 routing is enabled by default.
- IPv6 routing is **disabled** by default; must be enabled with `ipv6 unicast-routing`.
	- If IPv6 routing is disabled, the router will be able to send and receive IPv6 traffic, but will not *route* IPv6 traffic (= will not forward it between networks).
	- Troubleshooting: If everything is configured correctly but routing doesn't work, check this command out!

Example Topology used throughout this note:
![[Pasted image 20240903110815.png]]
## Routing table
R1's routing table before configuration:
![[Pasted image 20240903110909.png]]
- A Connected *network route* is automatically added for each connected network.
- A local *host route* is automatically added for each address configured on the router.
	- Host route: a route to a single specific host, using a `/32` prefix length on IPv4 or a `/128` prefix length on IPv6. 
- `FF00::/8` = IPv6 multicast range
	- `via Null0` = An interface that discards traffic matching the route; discards multicast traffic (Automatically configured) 
- Link-local addresses are not added to the routing table.
- Configuring network, host, and default routes:
	![[Pasted image 20240903114443.png]]
## IPv6 Static Route Configuration
![[Pasted image 20240903110815.png]]
`ipv6 route destination/prefix-length {next-hop | exit-interface [next-hop]} [ad]`
- `{next-hop | exit-interface [next-hop]}`: Required choice, either enter a next-hop address or an exit-interface with an optional next-hop.
- `[ad]`: Administrative distance is optional. (Used when configuring a floating static route)
- **Directly attached** static route: `ipv6 route destination/prefix-length exit-interface`
	- Only the exit interface is specified.
	- In **IPv6**, you **CAN'T use** directly attached static routes if the interface is an **Ethernet interface.**
	- E.g., This command WON'T WORK on R1: `ipv6 route 2001:db8:0:3::/64 g0/0`
- **Recursive** static route: `ipv6 route destination/prefix-length next-hop`
	- Only the next-hop is specified.
	- E.g., R1 uses this command: `ipv6 route 2001:db8:0:3::/64 2001:db8:0:12::2`
		- R3's LAN is the destination.
		- Tells R1 to send the packet to R2.
	- Requires a recursive lookup on the routing table; R1 has to check its routing table multiple times.
		1. Look up destination.
		2. Look up the next-hop to check which interface to send traffic out of.
	- **Link-local addresses cannot be used as the next hop** of an IPv6 static route!
		- Both the next hop address and the exit interface has to be specified. (Use the fully specified static route)
		- The router doesn't know which interface the link-local address is connected to.
- **Fully specified** static route: `ipv6 route destination/prefix-length exit-interface next-hop`
	- Both the exit interface and the next-hop are specified.
	- E.g., `ipv6 route 2001:db8:0:3::/64 g0/0 2001:db8:0:12::2`
`ipv6 route ::/0 DEFAULT_ROUTE`
- Configures the default route (in the recursive static route format)
- E.g., Configuring the default route from R3 = `ipv6 route ::/0 2001:db8:0:23::1`
## IPv6 Floating Static Route Configuration
![[Pasted image 20240903122257.png]]
Increasing the administrative distance can make static backup routes.
- In Cisco IOS, a normal static route has an AD of 1, so make the AD higher than 1.
- E.g., `ipv6 route 2001:db8:0:3::/64 s0/0/0 FE80::20B:BEFF:FED7:4901 5` Makes the route through R2 a backup route.
- Both routes are added to the running config, but only the route R1-R3 is added to the routing table: ![[Pasted image 20240903122450.png]]![[Pasted image 20240903122401.png]]