A protocol used in IPv6. It has various functions, and one of them is to replace ARP, which is no longer used in IPv6.
- The ARP-like function of NDP uses ICMPv6 and solicited-node multicast addresses to learn the MAC address of other hosts.
	- ARP in IPv4 uses broadcast messages.
	- Solicited-node multicast addresses are more efficient, since messages are addressed to specific hosts.
- Two message types are used:
	1. Neighbour Solicitation (NS) = ICMPv6 Type 135
		- NDP equivalent of ARP request

12:52
