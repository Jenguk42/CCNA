INCOMPLETE! [[https://youtu.be/4YrYV2io3as?si=Wxza_YefH-TKPZQK|Life of a Packet by Jeremy's IT Lab]]
### PC1 Sending Packet to PC4

![[Pasted image 20240815152358.png]]
-  Following IP addresses are encapsulated in the header.
	![[Pasted image 20240815153600.png]]
	- IP addresses: Layer 3 destination
	- MAC addresses: Layer 2 destination
	- Packet is sent to R1 (default gateway), since PC1 knows that the dest. IP of the is in a different network
- Note:
	- In the IPv4 header, the **source IP address** comes first; but in the ethernet header, the **destination MAC address** comes first.
	- The source & destination IP addresses of a packet do not change, but the MAC addresses change each time the packet passes through a router.
		- The switch will not alter any IP address, it will forward or flood it. (Refer to [[MAC address]]) 
	- To check the MAC address of a Windows computer, type `ipconfig /all` in the CLI
### PC1 to R1
1. PC1 sends an ARP request to R1 
	![[Pasted image 20240815152430.png]]
	- Dst MAC address is the broadcast MAC address (`FFFF.FFFF.FFFF`), because it does not know the MAC address of R1
2. SW1 receives the frame and broadcasts out of all its interfaces except the one i
3. t received the frame from.
	 - SW1 learns PC1's MAC address on its G0/1 interface via Src IP
4. R1 receives the frame from PC1 and de-encapsulates it (remove L2 header/trailer) to look at the inside packet.
	- If the routing table is not updated:
		- R1 has no matching routes in the routing table, so it will drop the packet
	- Refer to [[Static Routing]] to check