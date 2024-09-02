## Redundancy in Network
Redundancy is an essential part of network design to make sure the infrastructure is resilient to failures.
- Modern networks are expected to run 24/7/365, even a short downtime can be disastrous for a business.
- If one network component fails, there should be an alternate path enabled.
	- Ensure that other components will take over with little or no downtime!
- You must implement redundancy at every point possible, as much possible in the network.
- Important servers have multiple **NIC(Network Interface Card)s**, which can be plugged into multiple switches.
## Problems from Redundant Paths
### Broadcast Storms
Network congestions can happen when looped broadcasts accumulate in the network. This may prevent legitimate traffic use in the network.
1. A PC sends an ARP request frame, which is a broadcast frame with a destination MAC address with FFFF.FFFF.FFFF.
2. Switches will flood it out of all interfaces, except the one it was received on.
3. the frame and forward them from all interfaces other than the one it received the frame from.
4. The switches keep flooding the broadcast frames, which will lead to infinite loops. ![[Pasted image 20240820102331.png]]
	- TTL field of the IP header prevents infinite loops at layer 3, but the Ethernet header does not have a TTL field!
### MAC Address Flapping
When frames with the same source MAC address repeatedly arrive on different interfaces (due to broadcast storms), the switch continuously updates its interface in its MAC address table.

A Layer 2 protocol STP (Spanning Tree Protocol) can be used to enable network redundancy in layer 2 networks, while solving these problems.
## Spanning Tree Versions
![[Pasted image 20240821140908.png]]
- IEEE:
	- [[Classic STP (Spanning Tree Protocol), IEEE 802.1D]]
	- [[RSTP (Rapid Spanning Tree Protocol), IEEE 802.1W]]
	- MSTP (Multiple Spanning Tree Protocol), IEEE 802.1S: Useful when grouping large number of VLANs, industry standard for large networks
- Cisco Versions:
	- [[PVST+ (Per-VLAN Spanning Tree Plus)]]
		- Common for small to medium networks
		- 802.1W does not run on Cisco Switches
		- Works the same as RSTP, with the addition of separate instances for each VLAN.
	- No Cisco version of MSTP, Cisco uses the industry standards
## Port States 
![[Pasted image 20240820145801.png]]
- **Disabled State**: The interface is administratively disabled (shutdown) 
- Stable states - Ports remain stable within these states, as long as the network is stable
- Transitional states - Ports are open to changes
	- Each 15 sec long by default, determined by the **Forward delay** timer
		- Listening to Learning to Forwarding state: takes 30 secs

- **Forwarding State**: Maintained by Root/Designated ports
	- Operates as normal:
		- ✅Forwards/receives STP BPDUs
		- ✅Sends/receives regular network traffic
		- ✅Learns MAC addresses from regular traffic that arrives on the interface
	- ✅Can move directly into a blocking state
	
- **Listening State**: Interfaces with **Root/Designated** ports enter this state **after the Blocking state** (Non-designated ports are always blocking) 
	- Passed through when the blocking port must transition to a forwarding state due to network topology 
	- ✅ONLY forwards/receives STP BPDUs
	- ❌Does NOT send/receive regular network traffic (If a frame arrived, just drop)
	- ❌ Does NOT Learn MAC addresses from regular traffic that arrives on the interface
		
- **Learning State**: Interfaces with **Root/Designated** ports enter this state **after the Listening state** (Non-designated ports are always blocking) 
	- Passed through when an interface is activated
	- ✅ONLY forwards/receives STP BPDUs
	- ✅**LEARNS MAC addresses** from regular traffic that arrives on the interface!
		- Preparing to forward traffic by building up some MAC address table beforehand
	- ❌Does NOT send/receive regular traffic
		
- **Blocking State**: Maintained by Non-designated ports
	- Redundant interfaces are effectively disabled to prevent loops
	- Receive STP BPDUs, to be aware of the spanning tree topology and be ready for a transition towards forwarding state 
	- ❌ Does NOT send/receive Regular network traffic (If a frame arrived, just drop)
	- ❌ Does NOT Forward STP BDPUs
	- ❌ Does NOT Learn MAC addresses
	- ❌ Can NOT move directly to forwarding state, should go through listening & learning
## Port Types
- **Designated ports** (forwarding state)
	- All interfaces connecting to an end host (PC, etc.) is safe to use a designated port, because there is no risk of creating a Layer 2 loop.
	- Every collision domain has a single STP designated port.
- **Root ports** (forwarding state)
	- Ports connected to another switch's root port MUST be designated.
	- The root port is the switch's path to the root bridge, which must not be blocked. 
- **Non-designated ports** (blocking state)
## BPDUs (Bridge Protocol Data Units)
Switches use BPDUs to advertise themselves to other switches, and to learn about other switches.
### Hello BPDU 
Hello BPDU is received on a switch interface ➡️ the interface is connected to another switch. (Routers, PCs, etc. do not use STP, so they do not send Hello BPDUs.)
-  Default timer: 2 seconds
1. If all switches come online at the same time, they all send BPDUs out of all interfaces assuming they are the root bridge. ![[Pasted image 20240820112947.png]]
2. Once the network has converged and all switches are stabilised, only the root bridge sends BDPUs. The other switches forward these BDPUs on their **designated ports**, updating information (bridge root cost, sending bridge ID, sending port ID, etc.) ![[Pasted image 20240820150346.png]]
3. 2 seconds later, the root bridge send the BPDU again and the other switches will forward them out of their **designated** ports. (Not root or non-designated ports!!)
## Timers
![[Pasted image 20240820145853.png]]
- **Max Age Timer**: Makes sure loops are not accidentally created by an interface moving to forwarding state too soon
	- If another BDPU is received before the max age timer counts down to 0, the time will reset to 20 secs and no changes will occur.
	- If another BDPU is not received, the max age timer counts down to 0, and the switch will re-evaluate its STP choices (root bridge, local root, designated & non-designated ports, etc.)
		- This will take 20 secs.
	- If a non-designated port is selected to become a designated/root port, following transitions will happen:
		- Blocking ➡️ Listening (15 sec) 
		- ➡️ Learning (15 sec) 
		- ➡️ Forwarding
		- It can take a total of 50 secs for a blocking interface to transition into forwarding.
- This wait time can be shortened through PortFast. (Refer to [[STP Toolkit (Optional Features)]])
## Bridge ID
### Field 1: bridge priority + extended system ID
![[Pasted image 20240820113838.png]]
![[Pasted image 20240820114300.png]]
- Default Bridge ID is 32769.
	- Default bridge priority is 32768.
	- Default VLAN ID is 1.
- The STP bridge priority can only be changed **in units of 4096**.
	- Bridge priority + extended system ID is a single field of the bridge ID.
	- Extended System ID cannot be changed, it is determined by the VLAN ID.
	- 4096 is the least significant bit of the bridge priority, so the bridge priority can be 32768 +- 4096
	- Valid values that can be configured: 0, 4096, 8192, … , 61440
- E.g., If bridge priority is 32768, bridge id for VLAN1: 32769, VLAN2: 32770, etc.
### Field 2: MAC Address of the switch
- Cisco's PVST+ uses the destination MAC address of `0100.0ccc.cccd` for its BPDUs.
- Regular STP (not Cisco's PVST+) uses dest MAC address of `0180.c200.0000`
## Root Costs
(How far is the switch from the root bridge?) Each outgoing interface has an associated spanning tree cost. All interfaces of the root bridge has a cost of 0. 
![[Pasted image 20240820115516.png]]
- Regular Ethernet interface (10 Mbps): 100
- Fast Ethernet interface (100 Mbps): 19
- Gigabit Ethernet interface (1 Gbps): 4
	- 10 Gigabit Ethernet interface (10 Gbps): 2
- E.g., Network topology with root costs ![[Pasted image 20240820115847.png]]
## Port ID
![[Pasted image 20240820120825.png]]
STP Port ID = Port priority (Default 128) + Port number
- Can be checked through command `show spanning-tree`.
- Default port priority is 128.
- Each interface has a unique port number. (E.g., `G0/0` is 1, `G0/1` is 2. etc.) 
## STP Process
There is a set process that STP uses to determine which ports should be forwarding and which should be blocking.
1. **ONE** switch is elected as the root bridge. **ALL** ports on the root bridge are **designated ports.**
	-  Root bridge selection: 
		1. Lowest bridge ID
	- If it's RSTP and there is a hub, the root bridge has only one designated port for each collision domain, so the other interfaces within the domain will be in a backup state.
2.  Each remaining switch will select **ONE** of its interfaces to be its **root port**. Ports across from the root port will be **designated ports**.
	-  Root port selection: 
		1. Lowest root cost (Check the connection, going through two gigabit interfaces 4+4 is cheaper than one ethernet interface 19)
		2. Lowest neighbour bridge ID
		3. Lowest NEIGHBOUR port ID (Not the local switch's port ID!)
3. Each remaining collision domain will select **ONE** interface to be a designated port. Ports across the collision domain will be **non-designated ports**.
	- Designated port selection:
		1. Interface on switch with lowest root cost
		2. Interface on switch with lowest bridge ID
Some examples:
![[Pasted image 20240820132551.png]]
![[Pasted image 20240820132533.png]]


