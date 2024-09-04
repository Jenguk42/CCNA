## Layer 2 Discovery Protocols
Share information with and discover information about neighbouring (connected) devices, only using the Layer 2 information (not IP addresses).
- Shared information: host name, IP address, device type, etc.
- Can be considered a security risk and are often not used (information about the devices is shared on the network) 
	- It's up to the network engineer/admin to decide if they want to use them in the network or not.
- Devices periodically exchange frames to share information
	![[Pasted image 20240904100002.png]]
	- Note that the IP address is not added, because the switch is a Layer 2 device.
## CDP (Cisco Discovery Protocol)
Cisco proprietary protocol, enabled on Cisco devices (routers, switches, firewalls, IP phones, etc.) by default.
- **CDPv2** messages are sent by default. (v1 is old, not used.)
	- v2 has more advanced features - e.g., ability to identify native VLAN mismatches.
- Sent once every **60 seconds**, by default.
- Sent to multicast MAC address `0100.0CCC.CCCC`.
- When a device receives a CDP message, it processes and discards the message. It does NOT forward it to other devices.
	- This means only directly connected neighbours can become CDP neighbours.
	- The message is processed by adding an entry for the device in the CDP neighbour table.
- CDP hold-time is **180 seconds** by default.
	- If a message is not received from a neighbour for 180 seconds, the neighbour is removed from the CDP neighbour table.
## LLDP (Link Layer Discovery Protocol)
Industry standard protocol (IEEE 802.1AB)