A protocol used in IPv6. It has various functions, and one of them is to replace ARP, which is no longer used in IPv6.
- The ARP-like function of NDP uses ICMPv6 and solicited-node multicast addresses to learn the MAC address of other hosts.
	- ARP in IPv4 uses broadcast messages.
	- Solicited-node multicast addresses are more efficient, since messages are addressed to specific hosts.
- Two message types are used:
	1. Neighbour Solicitation (NS) = ICMPv6 Type 135
		- NDP equivalent of ARP request
	2. Neighbour Advertisement (NA) = ICMPv6 Type 136
		- NDP equivalent of ARP reply
## Neighbour Solicitation (NS)
NDP uses **multicast**, unlike ARP using broadcast - this is more efficient.
![[Pasted image 20240903103224.png]]
R1 tries `ping 2001:db8::78:9abc`.
- R1 needs to encapsulate the packet in an ethernet frame, so it needs R2's MAC address.
- To ask for the MAC address, R1 sends a Neighbour Solicitation message.
	- R1: "Hi, what's your MAC address?"
	- Source IP: R1's G0/0 IP address
	- Destination IP: R2's solicited-node multicast address (Refer to [Solicited-node multicast address](<IPv6 Addressing#Solicited-Node Multicast Address>))
		- R1 automatically calculates the solicited-node multicast address from the unicast address typed in the `ping` command, which is `ff02::1:ff78:9abc`.
	- Source MAC: R1's G0/0 MAC address
	- Destination MAC: Multicast MAC based on R2's solicited-node multicast address
## Neighbour Advertisement (NA)
![[Pasted image 20240903103643.png]]
In response to R1's NS, R2 teaches R1 its MAC address.
- Source IP: R2's G0/0 IP address
- Destination IP: R1's G0/0 IP address
	- R2 knows R1's IP address because it was the source IP address of the NS message.
- Source MAC: R2's G0/0 MAC address
- Destination MAC: R1's G0/0 MAC address
	- R2 knows R1's MAC address because it was used as the source MAC address of the NS message.
## IPv6 Neighbour Table
All devices (not just Cisco routers) running IPv6 will use NDP and keep a neighbour table.
- `show ipv6 neighbor`: View the neighbour table created via NDP
![[Pasted image 20240903104225.png]]
## Automatic Discovery of Routes
NDP allows hosts to automatically discover routers on the local network. (RS and RA are not to be mixed up with the NS and NA!) 
- Two messages are used for this process:
	- **Router Solicitation (RS)** = ICMPv6 Type 133
		- Sent to multicast address `FF02::2` (all routers) - Received by all routers in the local link.
		- Asks all routers on the local link to identify themselves.
		- Sent when an interface is enabled/ host is connected to the network.
	- **Router Advertisement (RA)** = ICMPv6 Type 134
		- Sent to multicast address `FF02::1` (all nodes) - Received by all hosts in the local link.
		- Routers use RA to announce its presence, as well as other information about the link.
		- Sent in response to RS; If a router receives an RS, it will send RAs.
		- Sent periodically even if the router hasn't received an RS.
- Lots of ways it can be used:
	- Hosts can automatically learn their default gateway from the RA messages.
![[Pasted image 20240903105029.png]]
R2's G0/0 interface automatically sends RS when it's enabled, asking if there are any routers on the link. R1 replies, identifying itself.
## SLAAC (Stateless Address Auto-configuration)
Hosts use the RS/RA messages to learn the IPv6 prefix of the local link (i.e. `2001:db8::/64`), and then automatically generate an IPv6 address.
- Using the `ipv6 address PREFIX/PREFIX-LENGTH eui-64` command, you need to manually enter the prefix.
- However, using the `ipv6 address autoconfig` command, you don't need to enter the prefix since the device uses NDP to learn the prefix used on the local link.
- The device will then use EUI-64 to generate the interface ID (or randomly generate it, depending on the device & maker).
## DAD (Duplicate Address Detection)
Allows hosts to check if other devices on the local link are using the same IPv6 address.
- DAD is performed on a device any time when:
	- An IPv6-enabled interface initialises (`no shutdown`).
	- An IPv6 address is configured on an interface. (manual, SLAAC, etc.)
- Uses two messages: NS (Neighbour Solicitation) and NA (Neighbour Advertisement).
	- The host sends an NS to its own IPv6 address.
	- If it doesn't get a reply, it knows the address is unique.
	- If it does get a reply, it means another host on the network is already using the address.