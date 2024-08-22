### Switch Interfaces Configuration
* **`show ip interface brief`**
	![[Pasted image 20240813141501.png]]
	Cisco switch Interfaces are **NOT** administratively down by default 
	![[Pasted image 20240813154309.png]]
	- If connected to a device, it will be up/up by default
	- IP Address will remain unassigned, since these are layer 2 switch ports 
	- down/down means the interfaces are not connected to another device, not that they are shut down
- **`show interfaces status`**
	![[Pasted image 20240813154750.png]]
	Check **switch status** - CANNOT BE USED ON ROUTERS!
	* `Name`: Description of interface 
	* `Status`: Connected or not connected to an end host
	* `Vlan`: Used to divide LANs to smaller LANs
		* Refer to [[VLANs]]
	* **`Duplex`**: Indicates if the device is capable of both sending and receiving data
		* Refer to [[Hubs, Full & Half Duplex]]
		* **`auto`**: Will negotiate with the neighbouring device and use full-duplex if possible (default)
			* Refer to [[Speed, Duplex Autonegotiation]]
		* **`a-auto`**: Automatically negotiated a duplex of full with the neighbouring device
	- **`Speed`**: Capable of 100 Mbps/sec (100 megabits per sec, fast ethernet cable)
		- **`auto`**: Will negotiate with the neighbouring device and use the fastest speed both devices are capable of (default)
			- Refer to [[Speed, Duplex Autonegotiation]]
		- `**a-100**`: 100 Mbps/sec was auto negotiated with the neighbouring device
	- `Type`: Type of the cable used, `10/100` refers to the speed at which the interfaces operate
- **`speed, duplex`**
	![[Pasted image 20240813155253.png]]
	* These can be manually configured
* **`interface range RANGE_START - RANGE_END`***
	![[Pasted image 20240813155518.png]]
	(interfaces are administratively down)
	Brings you to interface range config mode (`config-if-range`)
	* Helps you select and configure multiple interfaces
	* Range can be selected like this too: `int range f0/5 - 6, f0/9-12
		* `f0/7` and `f0/8` will not be affected
* **`show interfaces`** 
	![[Pasted image 20240813200815.png]]
	Useful when detecting errors
	- Can see total number of packets received on the interface
	- **Runts**: Frames that are smaller than the minimum frame size (64 bytes)
	- **Giants**: Frames that are larger than the maximum frame size (1518 bytes)
	- **CRC**: Frames that failed the CRC check in the Ethernet FCS (Frame Check Sequence) trailer
	- **Frame**: Frames that have an incorrect format due to an error
	- **Input errors**: Total of various counters (such as the above four)
	- **Output errors**: Frames the switch tried to send, but failed to due to an error
### VLAN Commands
![[Pasted image 20240818212244.png]]
For detailed explanation of the concept refer to [[VLANs]].
- `show vlan brief`
	![[Pasted image 20240818200008.png]]
	- Displays the VLANs that exist on the switch, and which interfaces are in each VLAN
		- Shows the **access ports** assigned to each VLAN, NOT the **trunk ports** that allow each VLAN!
	- VLANs `1, 1002-1005` exist by default
		- Cannot be deleted!
		- All interfaces are in VLAN `1` by default
		- VLANs `1002` to `1005` are old technologies that are not used
- `show interfaces trunk`
	![[Pasted image 20240818221550.png]]
	- Used when confirming trunk ports!
	- `on`: The interface is manually configured as a trunk
#### VLAN Configuration
![[Pasted image 20240818182308.png]]
- `switchport mode access`
	- Set the interface as an access port
	- Switches connected to an end host should enter access mode by default, but it's a good idea to explicitly configure the setting and not rely on auto-negotiation
- `switchport access vlan VLAN_NUMBER`
	- Assign VLAN to the port
	- If VLAN does not exist, it will be created automatically when assigned
- `vlan VLAN_NUMBER`
	- Enter the configuration mode for the VLAN
	- Create the VLAN if it does not exist
- `name VLAN_NAME`
	- Do this under `config-vlan` (VLAN configuration mode)
	- Assign a name for the VLAN
### Trunk Configuration
- `switchport mode trunk`
	- Set the interface as a trunk port
- `switchport trunk encapsulation dot1q`
	![[Pasted image 20240818221503.png]]
	- If the switch supports both ISL and 802.1Q, the trunk encapsulation type is set to `Auto` by default.
	- To manually configure the interface as a trunk port, the encapsulation should be set to either 802.1Q or ISL.
	- If the switch only supports 802.1Q, this is not necessary.
- `switchport trunk allowed vlan COMMAND`
	![[Pasted image 20240818222006.png]]
	- Command to configure the VLANs allowed on a trunk 
	- `switchport trunk allowed vlan VLAN_ID`: Allow configuration of the list of VLANs allowed
		- E.g., `switchport trunk allowed vlan 10,30` allows VLANs 10 and 30 on the trunk![[Pasted image 20240818222324.png]]
	- `switchport trunk allowed vlan add VLAN_ID`: Adds allowed VLANs to the existing list
		- E.g., `switchport trunk allowed vlan add 20` adds VLAN 20 to the allowed list
		- VLAN 20 is not created yet, so it is not displayed in the VLANs allowed and active in management domain ![[Pasted image 20240818222428.png]]
	- `switchport trunk allowed vlan remove VLAN_ID`: Removes the VLAN from the existing list
		- E.g., `switchport trunk allowed vlan remove 20` removes VLAN 20 from the allowed list ![[Pasted image 20240818222625.png]]
	- `switchport trunk allowed vlan all`: Allow all VLANs on the trunk
		- E.g., `switchport trunk allowed vlan all` allows all VLANs from 1 to 4094 on the trunk
		- Same as the default state ![[Pasted image 20240818222807.png]]
	-  `switchport trunk allowed vlan except VLAN_ID`: Allow all VLANs except the ones that are specified
		- E.g., `switchport trunk allowed vlan except 1-5,10` removes VLANs 1 to 5 and 10 from the allowed list ![[Pasted image 20240818223209.png]]
	- `switchport trunk allowed vlan none`: Allow no traffic to pass over the trunk
		- Clears all VLANs from the allowed list ![[Pasted image 20240818223255.png]]
### Native VLAN Configuration
- `switchport trunk native vlan VLAN_ID`
	![[Pasted image 20240818223714.png]]
	- Allows to change the native VLAN to the given ID
### Inter-VLAN Routing via SVI
![[Pasted image 20240819143252.png]]
- `ip routing` (DO NOT FORGET!!)
	- Enables Layer 3 routing on the switch, and lets it build its own routing table
- `no switchport`
	- Configures the interface from a Layer 2 switchport to a Layer 3 router port
	- Now the interface can be used just like a router interface!
	- Commands like `show ip route`, `show ip interface brief`, and `show interface status` can be used
	- The configured interface will show `routed` ![[Pasted image 20240819143540.png]]
- `interface vlan VLAN_NUMBER`
	-  Creates an SVI for VLAN 10 and goes into configuration mode ![[Pasted image 20240819143820.png]]
	- Remember `no shutdown` - SVIs are shut down by default
- Conditions for the SVI to be up/up:
	1. The VLAN must exist on the switch. 
		- The switch will not automatically create the VLAN, unlike in access ports!)
	2. The switch must have at least one access port in the VLAN in an up/up state, AND/OR one trunk port that allows the VLAN that is an up/up state.
	3. The VLAN itself should not be shutdown
	4. The SVI must not be shutdown
## DTP (Dynamic Trunking Protocol)
- `show interfaces INTERFACE_ID switchport` ![[Pasted image 20240819202821.png]]
	- Shows the name, switchport, administrative mode, and operational mode
### DTP Configuration
![[Pasted image 20240819202414.png]]
- `switchport mode ?`
	- `dynamic`: Dynamically negotiate between access or trunk mode
- `switchport mode dynamic ?`
	- `auto`: 
	- `desirable`: Actively try to form a trunk with other Cisco switches
## STP (Spanning Tree Protocol)
Refer to [[STP (Spanning Tree Protocol) Fundamentals]] for explanation
![[Pasted image 20240820135614.png]]
- Show Commands
	- `show spanning-tree`
		- The cost displays the cost of this interface, not the total root cost ![[Pasted image 20240820135550.png]]
	- `show spanning-tree detail` 
		- Shows the cost of root path ![[Pasted image 20240820135941.png]]
	- `show spanning-tree summary` 
		- ![[Pasted image 20240820140023.png]]
- Configuration Commands
	- `spanning-tree mode ...`
		![[Pasted image 20240820160924.png]]
		- `pvst`: Classic spanning tree, 
		- `rapid-pvst`: RSTP 
	- `spanning-tree vlan VLAN_NO root primary`
		![[Pasted image 20240820161318.png]]
		- ONLY APPLIES TO VLAN 1! (Other VLANs may have different topology)
		- Manually configure the primary root bridge by manipulating the priority values
			- **Sets the STP priority to 24576.**
			- If another switch has a priority lower than 24576, it sets this switch's priority to **4096 less than the current lowest priority.**
	-  `spanning-tree vlan VLAN_NO root secondary` 
		![[Pasted image 20240820161426.png]]
		- ONLY APPLIES TO VLAN 1! (Other VLANs may have different topology)
		- Configure the **secondary** root bridge, which will be used when the current root bridge fails
			- - **Sets the STP priority to 28672.**
	- `spanning-tree vlan VLAN_NO ...`
		![[Pasted image 20240820162544.png]]
		- Should be run in interface configuration mode
		- Root cost and bridge priority can be manipulated to change the port selection process.
		- `cost`: Change an interface's per VLAN spanning tree path cost
		- `port-priority`: Change an interface's spanning tree port priority
### Portfast
- `spanning-tree portfast`
	![[Pasted image 20240820154856.png]]
	- Enables Portfast at an interface level
	- Will only work for an access port
- `spanning-tree portfast default`
	- Enables Portfast on all ACCESS PORTS (not trunk ports)
### BPDU Guard
- `spanning-tree bpduguard enable`
	![[Pasted image 20240820155358.png]]
	- Enables BPDU Guard at an interface level
- `spanning-tree portfast bpduguard default`
	- Include `portfast` when enabling globally!
	- Enables BPDU Guards on all **Portfast-enabled** interfaces.
- `shutdown` and `no shutdown` when enabling the interface again (do this after solving the problem!!)
## EtherChannel
![[Pasted image 20240822141109.png]]
### Layer 2 EtherChannel
- `show etherchannel summary`: Shows a list of the port-channel interfaces
	![[Pasted image 20240822135341.png]]
	- EtherChannel 1 is in use (U). It is in layer 2 (S), and it is properly bundled in the port-channel (P).
- `show etherchannel port-channel`: Shows the number of ports, protocol being used, etc.
	![[Pasted image 20240822135633.png]]
	- Channel group mode is not shown on the summary, only available by this command!
- `show etherchannel load-balance`: Show the current load-balancing method
	- Default for the below switch: source and destination IP addresses ![[Pasted image 20240822125946.png]]
	- Keyword for SHOWING: `etherchannel`
	- Frames that encapsulate IP packets (IPv4 and 6) will be load-balanced based on source & destination IP addresses.
	- If the IP packet is not encapsulated in ethernet frame, use MAC addresses instead.
	
- `port-channel load-balance MODE`
	![[Pasted image 20240822130209.png]]
	- Keyword for CONFIGURATION: `port-channel`
	- Choices of load balancing modes: ![[Pasted image 20240822130407.png]]

- `channel-group NUMBER mode {desirable|active|passive|on}`: Choose configuration mode
	![[Pasted image 20240822132505.png]]
	- Modes should not be mixed. (E.g., `auto` and `active` will not form an EtherChannel)
		![[Pasted image 20240822141611.png]]
		- PAgP: `auto`, `desirable`
			![[Pasted image 20240822133438.png]]
		- LACP: `active`, `passive`
			![[Pasted image 20240822134135.png]]
		- Static EtherChannel: `on`
			- Only works with `on` mode. (`active` or `desirable` will not form EtherChannel if connected to `on`).
	- A virtual port-channel interface is created, can be checked using `show ip interface brief`
		- Command says `channel-group`, but the name of the interface created is **port-channel.**
	- The channel-group number has to match for member interfaces on the **same** switch, but does not have to match the channel-group number on the neighbour switch.
		- E.g., Channel-group 1 on ASW1 can form an EtherChannel with channel-group 2 on DSW1.
- `channel-protocol PROTOCOL`
	![[Pasted image 20240822134608.png]]
	- Protocol choices: `lacp`, `pagp` 
	- This command manually configures the protocol. Wrong modes will not be configured, and the range command will terminate.
	- There is no need because the protocols are automatically configured with mode.
- `interface port-channel NUMBER`
	![[Pasted image 20240822134910.png]]
	- Enters port channel configuration mode
	- The above trunk configurations will be applied to the physical interfaces within the port channel.
### Layer 3 EtherChannel
Remember to use `no switchport` command to make the interfaces Layer 3.
![[Pasted image 20240822140641.png]]
Then configure an IP address to the EtherChannel.
![[Pasted image 20240822140753.png]]