Everything here is equivalent to Rapid PVST+!

Bridge-bridge **handshake mechanism**, which allows ports to move directly to forwarding
- "Not a 'revolution' of STP, just an 'evolution'".
- Takes less time than timer-based spanning tree algorithm like 802.1D (Ports take 30 to 50 seconds to react to a change and start forwarding traffic)

RSTP has high similarity with STP:
- Serves the same purpose as STP, blocking specific ports to prevent Layer 2 loops.
- Elects root bridge, root ports, and designated ports with the same rules as STP.
## Difference with STP
### Root Costs 
Accommodates interfaces with higher speed.
![[Pasted image 20240821144150.png]]
### Port States
Blocking & Disabled states are combined into the Discarding state, and the listening state is not used.
![[Pasted image 20240821144346.png]]
- Listening state is unnecessary, and the learning state is skipped due to the built-in features
- Discarding state:
	- Administratively disabled with `shutdown` command (previously disabled state)
	- Enabled, but blocking traffic to prevent layer 2 loops (previously blocking state)
- **ALL** switches **originate** and **send** their own BPDUs from their designated ports!
### Port Roles
- **Root port and designated port** remains **UNCHANGED** in RSTP.
- **Non-designated port** role is split into two separate roles, alternate port role and backup port role.
- **Alternate** Port Role
	- **Discarding** port that receives a superior BPDU from **another switch** (= blocking ports in classic STP)
		- Superior BPDU: lower root bridge ID, lower root cost, lower sender bridge ID, etc.
	- Example topology ![[Pasted image 20240821144653.png]]
		- SW3 is receiving superior BDPU from SW2. (Bridge ID of SW2 is lower than SW3.)
		- So, SW2's interface is designated and SW3's is an alternate port.
	- Functions as a **backup to the root port** - if the root port fails, the switch can immediately move the best alternate port to forwarding.
		- If SW3's root port fails, the alternate port is ready to immediately become the root port with no transitional states.
- **Backup** port role
	- **Discarding** port that receives a superior BPDU from **another interface on the same switch**. (This only happens when two interfaces are connected to the same collision domain via a hub.)
	- The interface with the lowest port ID will be selected as the designated port, and the other will be the backup port.
	- Example topology ![[Pasted image 20240821150559.png]]
		- When BPDUs are sent into this network, the BPDU sent out of SW2's designated port is flooded by the hub.
		- SW2 receives this BPDU by the backup port.
	- Functions as a **backup to the designated port** - if the designated port fails, the switch can immediately move the best backup port to forwarding.
		- If SW2's designated port fails, the backup port is ready to immediately become the designated port with no transitional states.
- Example Topology ![[Pasted image 20240821153430.png]]
	- SW3 has a lower root cost, so one of its interfaces should be designated.
		- G0/0 has the lower port ID, so it becomes the designated port in the collision domain.
	- SW3's G01 receives the superior BPDU, with the lower port ID, from the same switch. so it becomes a backup port.
	- SW4's G0/0 receives the superior BPDU from a different switch, so it becomes an alternate port.
### BPDU (Bridge Protocol Data Unit)
![[Pasted image 20240821154031.png]]
- Protocol version is 2
- BPDU type is 2
- All 8 bits of BPDU flags used 
	- Used in the negotiation process that allows RSTP to converge faster.
#### New Functions Incorporated
These functions were optional features of classic STP which is built into the RSTP/Rapid PVST+. They do not need to be configured in RSTP/Rapid PVST+.
- Purpose: To help blocking or discarding ports move to forwarding without delay
- **UplinkFast**: If the root port fails, the switch can  immediately move the best alternate port to forwarding.
- **BackboneFast**: If there is an indirect link failure (the switch experiencing the failure does not have a direct connection to the failed link), the Max Age Timers can be expired to rapidly forward the superior BPDUs.
- **PortFast**: Integrated via Edge link type
### Other Important Differences
- **ALL** switches **originate** and **send** their own BPDUs from their designated ports, every hello time (2 seconds).
- Switches 'age' the BPDU information more quickly. 
	- Classic STP waits 10 hello intervals (20 sec)
	- RSTP considers a neighbour lost if it misses 3 BPDUs (6 seconds), then will 'flush' all MAC addresses learned on that interface.
		- Switch: "I can't reach this neighbour anymore. I'll clear all entries for this interface in my MAC table." (then makes the other interface its root port)
		- After the MAC table is cleared, it will re-learn by flooding.
## RSTP Link Types
Not to be confused with the spanning tree port roles and port states. There are three different 'link types':
1. **Edge**: A port that is connected to an end host. Moves directly to forwarding, without negotiation. (=PortFast)
	- Can move straight to forwarding state without the negotiation process
	- Functions like a classic STP Port with PortFast enabled
	- Enable in interface configuration mode by `spanning-tree portfast`
	- Edge ports can also be point-to-point or shared ports, depending on the full-duplex or half-duplex mode.
1. **Point-to-point**: A direct connection between two **switches**.
	- Functions in Full-duplex mode
	- No need to configure, it will be detected.
	- Explicit configuration: `spanning-tree link-type point-to-point`
2. **Shared**: A direct connection to a **hub**. (Hubs are old technology, not common)
	- Functions in Half-duplex mode
	- No need to configure, it will be detected.
	- Explicit configuration: `spanning-tree link-type shared`
## Example Topology
![[Screenshot 2024-08-22 101059.png]]