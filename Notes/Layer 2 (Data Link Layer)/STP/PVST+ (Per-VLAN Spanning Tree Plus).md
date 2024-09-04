Cisco switches uses a version of STP called **PVST (Per-VLAN Spanning Tree)**, which runs a separate STP 'instance' in each VLAN.
- Different setting for different VLAN - different interfaces can be forwarding/blocking in each VLAN.
- All configurations are done per-VLAN basis!
## Load Balancing
STP load balancing can be achieved by configuring a different root bridge for different VLANs, disabling different interfaces per LAN and balancing the network traffic.
- Manually configure primary and secondary roots, as well as cost and port priority of each interface.
	- Enable PVST: `spanning-tree mode pvst`
	- Configure primary, secondary root per VLAN: `spanning-tree vlan VLAN_NO root primary`, `spanning-tree vlan VLAN_NO root secondary`
	- Configure cost and port-priority per interface per VLAN: `spanning-tree vlan VLAN_NO cost`, `spanning-tree vlan VLAN_NO port-priority`
- If you have multiple VLANs in your network, blocking the same interface in each VLAN is a waste of interface bandwidth.