### Portfast
Allows a port to move immediately to the **Forwarding** state, bypassing **Listening** and **Learning.**
- Useful when trying to connect to an end host running quickly without waiting.
- Even when connected to an end host, the port goes through listening ➡️ learning ➡️ forwarding state. (This takes 30 seconds)
	- This is not necessary because Layer 2 loops will only form between switches, not with end hosts.
- Should only be enabled to ports connected to end hosts.
	- If enabled on a port connected to another switch, could cause a Layer 2 loop
	- Risk: what if a switch is connected instead of an end host? (Network cabling is changed without proper caution)
- Interface-level enable command: `spanning-tree portfast`
- To enable on all access ports: `spanning-tree portfast default`
### BPDU Guard
If an interface with BPDU Guard enabled receives a BPDU from another switch, the interface will be **shut down** to prevent a loop from forming.
- Prevents Layer 2 loops when using Portfast
- Interface-level enable command: `spanning-tree bpduguard enable`
- To enable globally: `spanning-tree portfast bpduguard default`
### Root Guard
When enabled on an interface, the switch will not accept a new switch as the root bridge even if it receives a superior BPDU (lover Bridge ID) on the interface. The interface will be disabled.
- Helps maintain the spanning tree topology with interference.
### Loop Guard
When enabled on an interface, the switch will not start forwarding even if the interface stops receiving BPDUs. The interface will be disabled.
- Prevents loops that can happen when an interface fails only in one direction.
- Will create a unidirectional data where data cannot be sent but can be received, or vice versa