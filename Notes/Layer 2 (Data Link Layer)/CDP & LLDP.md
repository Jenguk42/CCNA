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
- Periodically sent to multicast MAC address `0100.0CCC.CCCC`.
- When a device receives a CDP message, it processes and discards the message. It does NOT forward it to other devices.
	- This means only directly connected neighbours can become CDP neighbours.
	- The message is processed by adding an entry for the device in the CDP neighbour table.
- CDP hold-time is **180 seconds** by default.
	- If a message is not received from a neighbour for 180 seconds, the neighbour is removed from the CDP neighbour table.
### CDP Show Commands
![[Pasted image 20240904133731.png]]
![[Pasted image 20240904133508.png]]
![[Pasted image 20240904120305.png]]
`show cdp`
- Shows: CDP timer (60 seconds by default), CDP hold-time (180 seconds by default), version (v2 by default)
- `CDP is not enabled` will show if CDP is not enabled.
`show cdp traffic`
- Shows: How many CDP advertisements the device has sent & received.
![[Pasted image 20240904132524.png]]
`show cdp interface`
- Shows basic information about each interface.
- `Encapsulation ARPA`: ARPA is a type of ethernet encapsulation.
- Also shows how many interfaces are CDP enabled, and which are in an up state and a down state.
![[Pasted image 20240904132542.png]]
`show cdp neighbors`
- `Device ID`: Host name of R1's CDP neighbours
- `Local Intrfce`: Interface on the LOCAL device (R1), shows which interface is connected to which device
- `Holdtime`: Resets to 180 each time R1 receives a CDP message (With the default timer, it should be reset at 120)
- `Capability`: Identifies what kind of device you are connected to 
	- SW1 is a router AND a switch because it's a multilayer switch with routing capabilities
- `Platform`: Model of the neighbouring device
- `Port ID`: Port ID of the NEIGHBOURING device.
`show cdp neighbors detail`
- Shows more detailed information of the neighbours, including the IP addresses.
`show cdp entry NEIGHBOUR_HOSTNAME`
- Output is the same as `show cdp neighbours detail`.
### CDP Configuration Commands
CDP is globally enabled by default. Each interface is also enabled by default.
`[no] cdp run`
- Run on global config mode, to enable/disable CDP globally.
`[no] cdp enable`
- Run on interface config mode, to enable/disable on specific interfaces.
`cdp timer SECONDS`
- Run on global config mode, to configure the CDP timer.
`cdp holdtime SECONDS`
- Run on global config mode, to configure the CDP hold-time.
`[no] cdp advertise-v2`
- Run on global config mode, to enable/disable CDPv2.
## LLDP (Link Layer Discovery Protocol)
Industry standard protocol (IEEE 802.1AB), usually disabled on Cisco devices by default.
- A device can run CDP and LLDP at the same time, although usually only one is used.
- Sent once every **30 seconds**, by default.
- Periodically sent to multicast MAC address `0180.C200.000E`.
- When a device receives an LLDP message, it processes and discards the message. It does NOT forward it to other devices.
	- This means only directly connected neighbours can become LLDP neighbours.
	- The message is processed by adding an entry for the device in the LLDP neighbour table.
- LLDP hold-time is **120 seconds** by default.
- LLDP has an additional timer - 'reinitialization delay'.
	- If LLDP is enabled (globally/ on an interface), this timer will delay the actual initialization of LLDP.
	- **2 seconds** by default.