For VLAN Configuration on a switch, refer to [[Switch CLI Commands]]

![[Pasted image 20240818212816.png]]
### Purpose of VLANs
LANs can be split up into VLANs due to performance & security concerns.
- **Performance**: Lots of unnecessary broadcast traffic can reduce network performance.
	- VLANs help reduce unnecessary broadcast traffic, which helps prevent network congestion and improve network performance.
- **Security**: Even within the same office, you want to limit who has access to what.
	- Security policies can be applied on a router/firewall.
		- But if PCs are within the same LAN, they can directly reach each other without traffic passing through the router.
		- No effect of configuring security policies!
	- VLANs help limit broadcast and unknown unicast traffic, which improves network security.
		- You should always minimise unnecessary network traffic reaching other devices!
### What are VLANs?
A way to logically separating a Layer 2 broadcast domain to make multiple separate broadcast domains
- Configured on switches on a **per-interface** basis.
- **Logically** separates end hosts at Layer 2.
- Switches do not forward traffic directly between hosts in different VLANs.
### Access Ports & Trunk Ports
- Access port
	- = Untagged port (The interface belongs to a single VLAN)
	- A switchport which belongs to a single VLAN, and usually connects to end hosts like PCs
	- Gives end hosts access to the network
- Trunk port
	- = Tagged port
	- Switchports that carry traffic from multiple VLANs over a single interface
	- When a traffic passes through a trunk port, the switch determines which VLAN the traffic belongs to via **VLAN tagging**.
		- Switches will 'tag' all frames that they send over a trunk link.
	- Improves both security and network performance
		- Security: Makes sure only traffic in the necessary VLANs can use the connection
		- Avoids unnecessary traffic, because broadcasts in other VLANs will not be sent over the trunk
	- Check [[Switch CLI Commands#Trunk Configuration]] for trunk configuration
### VLAN Tagging
- Two main trunking protocols: ISL (Inter-Switch Link) and IEEE 802.1Q (dot1q).
	- ISL is an old Cisco proprietary protocol created before IEEE 802.1Q. (NOT USED!)
	- IEEE 802.1Q is an industry standard protocol created by the IEEE (Institute of Electrical and Electronics Engineers).
- 802.1Q tag is inserted between the source MAC address and the Type field
	![[Pasted image 20240818213600.png]]
	- Refer to [[Ethernet LAN Switching]] for the other parts of the frame
	![[Pasted image 20240818213653.png]]
	- 4 bytes (32 bits) in length
	- **Tag Protocol Identifier (TPID):** 16 bits
		- Fixed value of `0x8100` 
		- `0x` is hexadecimal, so the value is `8100` (4 * 4 = 16 bits)
		- Indicates that the frame is 802.1Q-tagged
	- **Tag Control Information (TCI)**: 16 bits
		- **Priority Code Point (PCP)**: 3 bits
			- Used for Class of Service (CoS), which prioritises important traffic in congested networks
		- **Drop Eligible Indicator (DEI)**: 1 bit
			- Used to indicate frames that can be dropped if the network is congested 
		- **VLAN ID (VID)**: 12 bits
			- Identifies the VLAN the frame belongs to
			- 12 bits in length = 4096 (2^12) total VLANS, range of 0 to 4095
			- VLANs 0 and 4095 are reserved, cannot be used
- Wireshark capture example
	- With dot1q tag![[Pasted image 20240819140158.png]]
	- Without dot1q tag (native VLAN function used)![[Pasted image 20240819140315.png]]
- VLAN Ranges
	- Range of VLANs (1 to 4094) is divided into two sections
		- Normal VLANs: 1 - 1005
		- Extended VLANs: 1006 - 4094
	- Modern switches support the extended VLAN range
### Native VLAN
- Feature in 802.1Q (ISL does not have it)
- If a frame is in the native VLAN, the switch does not add an 802.1Q tag.
	- If a switch receives an untagged frame on a trunk port, it assumes the frame belongs to native VLAN.
- Native VLANs should match between the connected switches!
	- If not the VLANs do not match, the switch will drop the frame
	- E.g., SW1 sends a tag-less frame in VLAN 10, which is its native VLAN.
	- SW2 receives the tag-less frame. However, its native VLAN is set as VLAN 30. Since the frame's destination is in VLAN 10 and not VLAN 30, it will drop the frame. 
- VLAN 1 by default on all trunk ports, but can be manually configured 
- It is best to change the native VLAN to an **unused VLAN** for security purposes
	- Limit unnecessary traffic in the network, and control what traffic is allowed
### Inter-VLAN Routing
Ways to route between VLANs
1. Use **one connection for each VLAN** between the router and switch
	- Works, but you may not enough interfaces on the router
2. **Router on a Stick (ROAS)**: use a single trunk connection which carries traffic from all VLANs between the switch and router
	- Efficient in terms of the number of interfaces
	- May cause network congestion in a busy network, since all traffic will go into the router and back to the switch
3. Layer 3 (Multilayer) Switch
### Router on a Stick (ROAS)
Used to route between multiple VLANs using a single physical interface on the router and switch.
![[Pasted image 20240818224534.png]]
![[Pasted image 20240818224731.png]]
- The router interface has to be configured using **subinterfaces**.
	- A VLAN tag and an IP address should be configured on each subinterface. (Refer to [[Router CLI Commands#VLAN Configuration]])
	- E.g., `G0/0` can be logically divided into three sub-interfaces - `G0/0.10`, `G0/0.20`, and `G0/0.30`
- The connected switch just needs to be configured like a regular trunk 
### Layer 3 (Multilayer) Switch
Capable of both switch AND routing, it is 'Layer 3 aware'.
- **SVIs (Switch Virtual Interfaces)** are virtual interfaces you can assign IP addresses to in a multilayer switch.
	- Not physical interfaces, but virtual interfaces in the software that can be used to route traffic at Layer 3.
	- Configure each PC to use the SVI (Not the router) as their gateway address. 
- To send traffic to different subnets/VLANs, the PCs will send traffic to the switch, and the switch will route the traffic.
- Can configure routes, like a router.
	- Has its own routing table
- Can be used for inter-VLAN routing.
- Example: ![[Pasted image 20240819142442.png]]
	- R1 and SW2 are connected with a point-to-point layer 3 link (no VLANs)
- The PCs can also connect to the cloud through R1 ([[Static Routing]]) ![[Pasted image 20240819142627.png]]