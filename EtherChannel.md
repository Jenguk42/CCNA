## Problems of Spanning Tree
**Oversubscription**: Bandwidth of the interfaces connected to end hosts > bandwidth of the connection to the distribution switches
- Some is acceptable, but too much will cause congestion.
![[Pasted image 20240822112051.png]]
Network Admin: "The connection to DSW1 is congested (there is oversubscription), so I should **add more links** to increase the bandwidth. Then all end hosts can be supported."
- But this will not work!
![[Pasted image 20240822112458.png]]
If you connect two switches together with multiple links, all except one will be disabled by spanning tree.
- This is to prevent Layer 2 loops and broadcast storms.
- Other links will be unused unless the active link fails. (Then one of the inactive links will start forwarding.)
This is a waste of bandwidth, because three links are not forwarding any traffic.
## Solution: EtherChannel
EtherChannel groups multiple interfaces together to act as a single interface, providing both redundancy and increased bandwidth. ![[Pasted image 20240822113015.png]]
- Four physical interfaces are formed into one logical interface.
- STP treats this group as a single interface.
	- Traffic using the EtherChannel will be load balanced among the physical interfaces in the group.
	- Can be checked through `show spanning-tree`
- There will be no Layer 2 loops, because the groups behave as if it is a single link.
	- If a PC sends a broadcast frame, DSW1 will only receive 1 copy.
	- It will not flood it to ASW1, because it received the frame from the same "single link" interface between them. 
- Up to 8 interfaces can be formed into a single EtherChannel.
	- LACP allows up to 16; 8 active, 8 standby (waiting for an active interface to fail)
- Some other names:
	- Port Channel
	- LAG (Link Aggregation Group)
## EtherChannel Load-Balancing
![[Pasted image 20240822113748.png]]
- EtherChannel load balances based on '**flows**'.
	- Flow: A communication between two nodes in the network
	- Frames in the same flow will be forwarded using the same physical interface. (If not, some frames may arrive out of order, which can cause problems.)
- EtherChannel has an algorithm that calculates **which physical interface should be used**.
	- Calculation Inputs that can be used:
		- Source MAC address
		- Destination MAC address
		- Source **AND** destination MAC addresses
			- E.g., Frames from PC1 to SRV1 should always use a certain interface.
		- Source IP address
		- Destination IP address
		- Source AND destination IP addresses
	- Different switch models have different methods.
- Load-balancing configuration
## EtherChannel Configuration 
### Methods
Three main methods of EtherChannel configuration on Cisco switches. Refer to [[Switch CLI Commands#Layer 2 EtherChannel]] for configuration commands.
- Similar to trunk formation in DTP (Dynamic trunk protocol)
- **LACP (Link Aggregation Control Protocol)** 
	- Industry standard protocol (IEEE 802.3ad), can be used on all switches (PREFERRED!)
	- Dynamically negotiates the creation/maintenance of EtherChannel.
	- Configuration: in interface range
- **PAgP (Port Aggregation Protocol)**
	- Cisco proprietary protocol, can only be used on Cisco switches
	- Dynamically negotiates the creation/maintenance of EtherChannel.
- **Static EtherChannel**
	- A protocol is not used, interfaces are statically configured to form an EtherChannel.
	- You want the switches to dynamically maintain the EtherChannel. (AVOIDED!)
		- E.g., If an interface is malfunctioning, you want the switch to remove it from EtherChannel dynamically.
### Rules for Member Interfaces
- Must have matching configurations!!
	- Same duplex (full/half)
	- Same speed
	- Same switchport mode (access/trunk)
	- Same allowed VLANs/native VLAN (for trunk interfaces)
- If an interface's configurations do not match the others, it will be excluded from the EtherChannel.
## Layer 3 EtherChannel
Layer 2 loops can still form even when using EtherChannels. For example:
![[Pasted image 20240822140400.png]] ![[Pasted image 20240822140303.png]]
Broadcasts can still loop around the switches and cause a broadcast storm. One interface has to be blocked to make sure there is no loop forming.
![[Pasted image 20240822140003.png]]
If all connections are made using routed ports, not Layer 2 switch ports, **there is no need to run spanning-tree at all.**
- Layer 2 loops will not be formed because router ports don't forward Layer 2 broadcasts.

Refer to [[Switch CLI Commands#Layer 3 EtherChannel]] for configuration commands.