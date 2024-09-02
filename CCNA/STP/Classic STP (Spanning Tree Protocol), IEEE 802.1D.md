Refer to [[STP (Spanning Tree Protocol) Fundamentals]] for detailed explanation.

STP Prevents Layer 2 loops by placing redundant ports in a blocking state, disabling the interface.
- Switches from ALL vendors run STP by default. (Not just on Cisco Switches)
- Interfaces in a **blocking** state:
	- Act as backups that can enter a forwarding state if an active (=currently forwarding) interface fails.
	- Only send and receive STP messages called **BPDUs (Bridge Protocol Data Units)**.
	- Bridge is an old technology used between hubs and switches, but in this context think of it as a switch.
- Interfaces in a **forwarding** state:
	- Behave normally, sending and receiving all normal traffic.

Example Topology: ![[Pasted image 20240820104412.png]]
- SW3 has an interface in a blocking state, which disconnects SW2 and SW3.
- If an interface in SW2 fails, the switches will automatically adjust the topology.
## BPDU (Bridge Protocol Data Unit)
 ![[Pasted image 20240820152829.png]]
- Protocol version is 0
- BPDU type is 0
- 2 bits of BPDU flags used (1st and 8th)
