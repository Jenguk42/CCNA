## Fundamentals
![[Pasted image 20240813131057.png]]
* `?` : View help
	* **No** space - Show all possible completions of the word
	```
	Router(config)#password?
	password
	```
	* **With** space - All possible options I can enter next in the command
	```
	Router(config)#enable password ?
	    7 Specifies a Hidden password will follow
	    LINE The UNENCRYPTED (cleartext) 'enable' password
	    level Set exec level password
	```
- `tab`: Let CLI complete the command
	* If there are multiple commands that begin with the given letters, CLI returns `% Ambiguous command: "(incomplete)"`
* `Ctrl + A` takes you to the front of the line
* `no COMMAND`: Remove the given command
	* E.g., `no service password-encryption`: future commands will not be encrypted
### Privileged EXEC Mode
- ==`enable`==: Enter privileged EXEC mode
	* `#` appears next to the router name (instead of `<`, which comes with the user EXEC mode)
	* More options such as changing configuration, deleting file, etc.
- ==`conf t`==: Enter global configuration mode
	* `(config)` appears next to the router name
- ==`do`== `PRIVILEGED-EXEC-LEVEL-COMMAND`: Execute privileged exec mode commands in global configuration mode
- ==`show running-config`==, ==`show startup-config`==: Display the configuration file
	* `Running-config`: The current, active configuration file on the device. You are editing the active configuration as you enter commands in the CLI
	* `Startup-config`: The configuration file that will be loaded upon restart of the device
	* If config not saved yet, the router will always open a default configuration, and display `startup-config is not present`.
	* `show running-config | include EXACT_LINE` shows the line that includes the given line from the running configuration
- ==`write`==, ==`write memory`==, ==`copy running-config startup-config`==: Save the running configuration file as the start-up configuration
### Security Configurations
- `enable password PASSWORD`: Create an **unencrypted** password (case sensitive) for entering privileged EXEC mode
	* Password does not display as you type, but gets saved in the configuration file (security risk!)
	* Three consecutively wrong pw returns `Bad Secrets` and terminate the line
* `service password-encryption`: Encrypts the password in the configuration file
	* `enable password 7 08026F6028` shown in the config file
		* `7`: Cisco's proprietary encryption algorithm is used
	        * `08026F6028`: Encrypted password
	        * Not very secure - cisco type 7 password cracker can be found online
	-  Disabling the password encryption with `no` command will not remove already encrypted password, but future commands will not be encrypted
- `enable secret PASSWORD`: Create an **always encrypted** password (case sensitive) for entering privileged EXEC mode
	```
	enable secret 5 $1$mERr$YlCkLMcTYWwkF1Ccndtll.
	enable password 7 08026F6028
	```
	* `5`: MD5 type encryption, more secure than type 7
	* `$1$mERr$YlCkLMcTYWwkF1Ccndtll.`: Encrypted password
	* Both `enable password` and `enable secret` are saved, but `enable password` will be ignored.
	* `enable secret` always takes precedence over the `enable password`!
	* `service password-encryption` command has no effect on `enable secret` command
## IPv4 Address Configuration
* `show ip interface brief` 
	(Before IP configuration) ![[Pasted image 20240813120101.png]](After IP configuration)![[Pasted image 20240813130654.png]]
	Used to confirm the status & IP addresses of each interface on the device
	* ==`do sh ip int br`==
	* Method: How is the IP address assigned?
	* Status: Layer 1 status of the interface (Interface shut down? cable attached? etc.)
		* administratively down: Interface has been disabled with the 'shutdown' command
		* Cisco **router** interface status is administratively down by default
		* Cisco [[Switch CLI Commands]] are **NOT** administratively down by default 
	* Protocol: Layer 2 status of the interface (Is Ethernet functioning properly?)
		* If status is down, protocol will always be down
* `interface INTERFACE_NAME`
	![[Pasted image 20240813130144.png]]
	Used to enter interface configuration mode
	- `(config-if)` is short for global configuration - interface configuration mode 
	* `interface gigabitethernet 0/0` (==`int g0/0`==)
	* Should be done in `config` mode
	* You can directly switch from one interface to the other
		* You don't need to use `exit` to return to global mode before entering `interface g0/1`.
* `ip address IP_ADDRESS`
	![[Pasted image 20240813130525.png]]
	Used to configure the ip address 
	* `ip address 10.255.255.254 255.0.0.0` (= ==`ip add ...`==)
	* This is equivalent to `10.255.255.254/8`
* `no shutdown`
	![[Pasted image 20240813130552.png]]
	Used to enable the interface (= ==`no shut`==)
- `show interfaces INTERFACE_NAME`
	![[Pasted image 20240813131421.png]]
	* Shows Layer 1 and 2 information about the interface
	* Line 2: bia stands for burned in address (physical MAC address)
		* Listed twice because MAC address can be differently configured in the CLI
* `show interfaces description`
	![[Pasted image 20240813131506.png]]
* `description DESCRIPTION`
	Description for interfaces can be configured separately to add info ![[Pasted image 20240813131619.png]]
-  `default interface INTERFACE_NAME`
	Sets the interface to default configuration ![[Pasted image 20240819142938.png]]
## Routing
- `show ip route`
	![[Pasted image 20240814141327.png]]Used to view the routing table
	* **Local route**
		* A route to the **actual IP address** configured on the interface
		* has a /32 mask
	* **Connected route**
		* A route to the **network** the interface is connected to
		* has the actual netmask configured on the interface
### IPv4 Static Route Configuration
Refer to [[Static Routing]] for detailed explanation
![[Pasted image 20240814153912.png]]
- `ip route IP-ADDRESS NETMASK NEXT-HOP` (Preferred)
	- **Recursive** static route: Only the next-hop is specified, requires a recursive lookup on the routing table.
		- Configures the next-hop to reach the IP address
	- E.g., For R1 to reach PC4, it should hop to `G0/0` interface of R3 which has an IP address of `192.168.13.3`.
		- `ip route 192.168.4.0 255.255.255.0 192.168.13.3`

- `ip route IP-ADDRESS NETMASK EXIT-INTERFACE
	- **Directly attached** static route: Only the exit interface is specified.
		- Configures which interface to send the packet out of to reach the IP address
	- E.g., For R2 to reach PC1, it should send the packet out of the `G0/0` interface.
		- `ip route 192.168.1.0 255.255.255.0 g0/0`
	- Shown in the routing table as "directly connected", although it is not exactly directly connected
		- ![[Pasted image 20240815133432.png]]
		- If you only specify the exit-interface, the router will have to use ARP (Proxy ARP) for every destination it wants to reach, so it can be considered worse.
		- Stick to using the next-hop in your static routes!

- `ip route IP-ADDRESS NETMASK EXIT-INTERFACE NEXT-HOP`
	- **Fully specified** static route: Configures both next-hop and interface
	- E.g., For R2 to reach PC4, it should:
		- Send the packet out of the `G0/1` interface and 
		- Hop to `G0/0` interface of R4 which has an IP address of `192.168.24.4`.
		- `ip route 192.168.4.0 255.255.255.0 g0/1 192.168.24.4`

- `ip route 0.0.0.0 0.0.0.0 DEFAULT-ROUTE
	![[Pasted image 20240815134608.png]]
	- Configures the  default route
	- `*` means that this route is the candidate to become the router's default route.
	- Possible to have multiple candidates
### IPv4 Floating Static Route Configuration
- `ip route DESTINATION_IP SUBNET_MASK AD`
	![[Pasted image 20240822163957.png]]
	- Static route is configures as usual, but with a different administrative distance.
	- If you want to keep the dynamic route as default and floating static route as a backup, AD should be **higher than the main route**.
		- E.g.1, OSPF has an AD of 110, so set the floating static route with AD 111.
		- E.g.2, EIGRP has an AD of 90, so set the floating static route with AD 91.
	- In this case, the static route will not show up in the routing table because the OSPF route is favoured and written on the table.
## IPv6 Addressing
### Show Commands
- `show ipv6 neighbor`: View the neighbour table created via NDP
	![[Pasted image 20240903104225.png]]
	- `IPv6 Addresses`: Both global unicast address and link local address are added. The link-local address is automatically added.
	- `Age`: How long ago was traffic received from the neighbour, in minutes?
	- `Link-layer Addr`: Neighbour's MAC address
	- `Interface`: Interface this entry was learned on
- `show ipv6 interface brief`
	- Same as the IPv4 command, just use `ipv6`.
	- Shortened version of the address is displayed.
	- Each interface has two IPv6 addresses.
		- **Link-Local Addresses**: automatically configured on an interface when you configure an IPv6 address.
### Configuration Commands
R1 below has 3 interfaces, each connected to a different subnet. The company was assigned a /64 address block, and is using a last quartet of its prefix to make different subnets.
![[Pasted image 20240901151059.png]]
![[Pasted image 20240901151220.png]]
`ipv6 unicast-routing` **IMPORTANT!!**
- Allows the routers to perform IPv6 routing
- If this is not done, end hosts cannot reach hosts connected to the router's other interface.
#### Manual Configuration
`ipv6 address IPv6_ADDRESS/PREFIX_LENGTH`
- The address should be followed by a prefix length
- You can use a whole, partially shortened, or shortened address when configuring IPv6 to routers.
![[Pasted image 20240901151430.png]]
#### EUI-64 Configuration
`ipv6 address PREFIX/PREFIX_LENGTH eui-64`
- Configure an IPv6 address, where: 
	- The first 64 bits (network prefix) are configured as given, and 
	- The remaining 64 bits (interface identifier) are automatically generated using the EUI-64 format.
- E.g., `ipv6 address 2001:db8::/64 eui-64`
![[Pasted image 20240902100144.png]]
Original MAC address of `g0/0`: `0cf8 2236 8500`.
![[Pasted image 20240902100204.png]]
IPv6 address configured: `2001:db8::ef8:22ff:fe36:8500`.
- Network prefix: `2001:db8::`.
- Interface identifier generated using the EUI-64 conversion: `0ef8 22ff fe36 8500`. (Leading `0` is removed.)
![[Pasted image 20240902101504.png]]
Link local addresses use the same interface ID as the global unicast address that is configured, because both uses EUI-64 to generate the interface ID.
![[Pasted image 20240902110645.png]]
#### SLAAC Configuration
`ipv6 address autoconfig`
- Used in interface configuration mode.
- Configure the address without entering the prefix.
- The device uses NDP to learn the prefix used in the local link.
- Refer to [SLAAC (Stateless Address Auto-configuration)](NDP%20(Neighbour%20Discovery%20Protocol).md#SLAAC%20(Stateless%20Address%20Auto-configuration) (Neighbour Discovery Protocol)#SLAAC (Stateless Address Auto-configuration)>)
Configuration on a router
![[Pasted image 20240903110208.png]]
Configuration on a PC (Tick `Automatic` in the global settings tab)
![[Pasted image 20240903120647.png]]
You can check that the IPv6 address is automatically generated:
![[Pasted image 20240903121003.png]]
#### Static Route Configuration
![[Pasted image 20240903110815.png]]
`ipv6 route destination/prefix-length {next-hop | exit-interface [next-hop]} [ad]`
- `{next-hop | exit-interface [next-hop]}`: Required choice, either enter a next-hop address or an exit-interface with an optional next-hop.
- `[ad]`: Administrative distance is optional. 
	- Used when configuring a floating static route - Increasing the administrative distance can make static backup routes.
	- In Cisco IOS, a normal static route has an AD of 1.
- **Directly attached** static route: `ipv6 route destination/prefix-length exit-interface`
	- Only the exit interface is specified.
	- In **IPv6**, you **CAN'T use** directly attached static routes if the interface is an **Ethernet interface.**
	- E.g., This command WON'T WORK on R1: `ipv6 route 2001:db8:0:3::/64 g0/0`
- **Recursive** static route: `ipv6 route destination/prefix-length next-hop`
	- Only the next-hop is specified.
	- E.g., R1 uses this command: `ipv6 route 2001:db8:0:3::/64 2001:db8:0:12::2`
		- R3's LAN is the destination.
		- Tells R1 to send the packet to R2.
	- Requires a recursive lookup on the routing table; R1 has to check its routing table multiple times.
		1. Look up destination.
		2. Look up the next-hop to check which interface to send traffic out of.
	- **Link-local addresses cannot be used as the next hop** of an IPv6 static route!
		- Both the next hop address and the exit interface has to be specified. (Use the fully specified static route)
		- The router doesn't know which interface the link-local address is connected to.
- **Fully specified** static route: `ipv6 route destination/prefix-length exit-interface next-hop`
	- Both the exit interface and the next-hop are specified.
	- E.g., `ipv6 route 2001:db8:0:3::/64 g0/0 2001:db8:0:12::2`
`ipv6 route ::/0 DEFAULT_ROUTE`
- Configures the default route (in the recursive static route format)
- E.g., Configuring the default route from R3 = `ipv6 route ::/0 2001:db8:0:23::1`
## Dynamic Routing Configuration
### Commands used in Common
- `show ip route ...`
	- `IGP_NAME`: Filters routes by IGP protocols. (EIGRP, OSPF, etc.)
	- `CONNECTION_TYPE`: Filter routes by connection types (static, connected, etc.)
- `show ip IGP_NAME neighbors`: Shows the neighbours of the router in the given protocol, as well as the interface they are connected to.
	- Example: EIGRP
		![[Pasted image 20240827114933.png]]
- `show ip IGP_NAME topology`: Detailed information about the EIGRP routes the router has received (not just the routing table)
	- Example: There are two routes to `192.168.4.0/24`, but only one with the lower metric value is entered in the routing table. ![[Pasted image 20240827115435.png]]
- `show ip protocols`: Shows various stats
	- Example: RIP 
		![[Pasted image 20240823143643.png]]
- `maximum paths NUMBER`: Change the maximum paths that can be saved for the same destination. NEEDS TO BE EXECUTED IN PROTOCOL CONFIGURATION MODE!	![[Pasted image 20240823143929.png]]
- `distance NUMBER`: Change the administrative distance to make it favoured over other protocols. NEEDS TO BE EXECUTED IN PROTOCOL CONFIGURATION MODE! ![[Pasted image 20240823144145.png]]
- `network IP_ADDRESS`: "Activate IGP on interfaces with an IP address that falls under the given IP range."
	- Tells the IGP to look for any interfaces with an IP address contained in the range specified in the `network` command.
	- Activate IGP on the interface in the specified range.
	- The router will then try to become IGP neighbours with other same-protocol-activated neighbour routers.
	- `network 0.0.0.0 255.255.255.255` can activate the IGP on all interfaces (all interfaces have IP addresses in the `0.0.0.0/0` range).
		- Not recommended in real life, but handy in labs
- `clear ip IGP_NAME process`
	- Reset all processes of the protocol.
	- E.g., `clear ip ospf process` will reset the OSPF and shut all interfaces down. 
#### Passive Interface Configuration
- `passive-interface INTERFACE_ID`
	- E.g., `passive-interface g2/0`
	- **DONE IN IGP CONFIGURATION MODE, NOT ON THE INTERFACE!** (This is why the interface should be specified in the command.)
	- Used in RIP, EIGRP, and OSPF
#### Default Route Configuration
- `default-information originate`
	- Share the default route with the neighbouring same-protocol-enabled routers.
#### Loopback Interface Configuration
- `interface loopback LOOPBACK_INT_NO`: Enables a loopback interface on a router
	![[Pasted image 20240827102319.png]]
	- Done in the global config mode.
	- Configure an IP address, common to use `/32` mask (E.g., `ip add 1.1.1.1 255.255.255.255`)
#### RIP Configuration
![[Pasted image 20240823132552.png]]
- `router rip`: Enter RIP configuration mode
- `version 2`: Use RIP version 2 to allow VLSM and CIDR 
- `no auto-summary`: Use classless networks
	- `auto-summary` is on by default, and it automatically converts the networks (advertised by the router) to classful networks.
	- E.g., `172.16.1.0/28` attached to R1 is converted to class B network, advertised as `172.16.0.0/16`
#### EIGRP Configuration
![[Pasted image 20240823145628.png]]
- `router eigrp AS`: Enter EIGRP configuration mode
	- AS (Autonomous System) number must match between routers, or they will not form an adjacency and share route information.
- `no auto-summary`: Use classless networks.
- A **wildcard mask** can be used with the `network` command!
	- It will assume a classful address if you don't specify the mask. (E.g., `10.0.0.0` is assumed to be `10.0.0.0/8`)
	- Usually the wildcard mask will have the same prefix length as the interface itself.
- `eigrp router-id ROUTER_ID`
	![[Pasted image 20240823153041.png]]
	- Manual configuration of router ID (Highest priority)
##### Unequal-Cost Load-Balancing
![[Pasted image 20240827133924.png]]
- `variance MULTIPLIER`: Configure the variance to allow unequal-cost load-balancing
	- E.g., Variance 2 = Feasible successor routes with an FD up to 2 times the successor route's FD can be used to load-balance. 
	- 28672 \* 2 = 57344, and 30976 is less than 57344, so the route via R3 can now be used for load-balancing. ![[Pasted image 20240827132621.png]]
	- After configuration, both paths are added in the routing table. However, `G0/0` will carry more traffic because it's a faster path with a lower metric. ![[Pasted image 20240827133000.png]]
#### OSPF Configuration
For an OSPF demo, refer to the first 4 minutes of [Day 34 Lab, Step 1](https://youtu.be/sJ8PXmiAkvs?si=saHY_YhU1r4k9qGD); The lab itself is about configuring ACLs, but it shows a nice quick configuration.
##### OSPF Enable Commands
- `router ospf PROCESS_ID`
	- Enables OSPF protocol with the given process ID and enters OSPF configuration mode.
	- The OSPF process ID is locally significant, so they don't have to match between OSPF neighbours. A router can run multiple OSPF processes at once, and this ID is used to identify between them.
	- Unrelated to the area!
- `network IP_ADDRESS WILDCARD AREA_NO`
	- Same as the [[#Network Command]] explained above, but only activates OSPF on the interface in the specified `area`.
	- For single-area OSPF, the best practice is to use area 0.
	- E.g., `network 10.0.12.0 0.0.0.3 area 0`
- `ip ospf PROCESS_ID area AREA`
	- DONE IN INTERFACE CONFIGURATION MODE! 
	- Activates OSPF directly on an interface.
	- E.g., `ip ospf 1 area 0` in `g0/0` configuration enables the OSPF protocol on the interface.
- `passive-interface default`
	- DONE IN OSPF CONFIGURATION MODE!
	- Configures ALL interfaces as OSPF passive interfaces.
	- Interfaces can be configured back as active by: `no passive-interface INT_ID`
##### OSPF Show Commands
Example Network Topology Used: ![[Pasted image 20240828093505.png]]
- `show ip ospf database`
	- Shows the LSDB. ![[Pasted image 20240828093403.png]]
- `show ip ospf neighbor`
	- `FULL` state: R2 and R3 are fully connected OSPF neighbours.
	- Both R2 and R3 are `DR`s. 
	- `Dead Time` is 40 by default, but resets as soon as R1 receives a Hello packet from the neighbour. ![[Pasted image 20240828093447.png]]
- `show ip ospf interface`
	- You can specify the interface you want to check. 
	- Notice the cost of `G0/0` and `F1/0` are the same, because the reference bandwidth was not updated. ![[Pasted image 20240828093628.png]]
- `show ip ospf interface brief`
	![[Pasted image 20240829101945.png]]![[Pasted image 20240829092042.png]]
	- `Nbrs`: `F` indicates the number of full adjacencies, and `C` indicates the total counts of neighbours
		- R3 has a total of 3 neighbours: R2, R4 and R5.
		- R3 has 2 full adjacencies with R2 and R4.
	- `State`: Shows if the interface is DROther, DR, or BDR. 
##### OSPF Cost Configuration
- Reference Bandwidth Configuration: `auto-cost reference-bandwidth MEGABITS_PER_SECOND`
	- In OSPF configuration mode
	- Crucial to allow proper cost calculation!
- Manual Configuration: `ip ospf cost COST`
	- Changes the cost of individual interface, in interface configuration mode
	- RECOMMENDED OVER CHANGING BANDWIDTH!
- Interface Bandwidth Configuration: `bandwidth KB_PER_SEC`
	- In interface configuration mode
	- Bandwidth matches the interface speed by default, but changing the interface bandwidth **does not change the speed at which the interface operates!**
	- E.g., If you change the bandwidth of a Gigabit Ethernet interface to 100 mbps, it will still operate at 1 gbps. However, a bandwidth of 100 mbps will be used for OSPF's cost calculation.
	- NOT RECOMMENDED - Bandwidth value is used in other calculations too!
##### Serial Interface Configuration
- `clock rate SPEED_IN_BPS`
	- Used to specify the clock rate on the DCE.
	- Remember to add an IP address and use `no shutdown`.
	- Different from ethernet interfaces (they use `speed`!)
- `show controllers INT_ID`
	![[Pasted image 20240829140301.png]]
	- Used to identify which is DCE and which is DTE. 
- `encapsulation PPP`
	- Manually configure the encapsulation to PPP, since the default is HDLC.
	- Encapsulations should **match between both ends**, or the interface will go down!
##### OSPF Other Configuration
- `ip ospf priority PRIORITY`
	- Manually configure the priority between 0 to 255
	- Used when choosing DR & BDR within the subnet
- `ip ospf network TYPE`
	![[Pasted image 20240829110948.png]]
	- Manually configure the network type on an interface.
- `router-id A.B.C.D`
	- Manual configuration of router ID (Highest priority)
	- Note that you don't need to specify `ospf` unlike in EIGRP.
	- `clear ip ospf processes` is used to clear the current ID when changing the router IDs.
		- Done in privileged EXEC mode!
		- Used when there are other neighbours that are affected.
		- `no router-id` can be used if it has no other OSPF neighbours.
	- Remember **router IDs MUST be unique** for the OSPF to work!
- `shutdown`
	- Done from OSPF configuration mode.
	- Disables OSPF operation, without removing the OSPF configurations.
- `ip ospf hello-interval TIME_IN_SECONDS`, `ip ospf dead-interval TIME_IN_SECONDS`
	- Changes the hello and dead timers.
	- `no ip ospf hello-interval` can be used to change it to the default interval. 
- Adding authentication to OSPF
	- `ip ospf authentication-key PASSWORD`: Creates password
	- `ip ospf authentication`: Enables authentication
	- Done from the interface config mode
- `ip mtu MTU_IN_BYTES`
	- Default is 15,000
	- Done from the interface config mode
	- `no ip mtu` can be used to change it to the default setting. 
## VLAN Configuration
### Router on a Stick (ROAS)
![[Pasted image 20240818224731.png]]
![[Pasted image 20240818225147.png]]
- `interface INTERFACE_ID.SUBINTERFACE_NO`
	- E.g., `interface g0/0.20`
	- Enter subinterface configuration mode (`config-subif`)
	- The subinterface number is recommended to match the VLAN number to make it easier to understand.
- `encapsulation dot1q VLAN_NO`
	- Tells the router to treat any arriving frames tagged with the specific VLAN number as if they arrived on this subinterface
	- Remember to configure the IP address for the subinterface!
	- `encapsulation dot1q 10`
		- If a frame arrives tagged with VLAN 10, R1 will behave as if it arrived on interface `g0/0.10`.
		- If a frame is leaving the subinterface `g0/0.10`, R1 will tag the frame with VLAN 10.
	- Alternative: configure the IP address for the native VLAN on the router's physical interface (in this case, `g0/0`)
		- No need for `encapsulation dot1q VLAN_ID` command
- `ip address SUBINTERFACE_IPADDRESS`
	- Remember to assign the IP address!
- Confirmation:
	The physical interface (`G0/0`) has no IP address assigned, but each subinterface (`G0/0.10`, `G0/0.20`, `G0/0.30`) gets its own IP address.  ![[Pasted image 20240818225201.png]]
	Connected and local routes are added too! ![[Pasted image 20240818225231.png]]
## HSRP Configuration
Configuring R1 and R2 to use HSRP to provide a redundant default gateway address for the `172.16.0.0/24` subnet:
![[Pasted image 20240829212526.png]]
HSRP is configured directly on the interface.
- `show standby`
	![[Pasted image 20240830102506.png]]
- `standby GROUP_NUMBER ip IP_ADDRESS` 
	- Range of groups available: 0 to 255 (0 to 4095 in v2)
		- Rule of thumb: HSRP group number matches the VLAN number used for the subnet.
		- MUST: HSRP group numbers should match between two routers!
	- Configure the virtual IP used for the default gateway (same virtual IP is configured for both routers)
	- E.g., `standby 1 ip 172.16.0.254`
- `standby version 2`
	- Changes the version number
- `standby GROUP_NUMBER priority PRIORITY`
	- Priority value can range from 0 to 255.
	- Used to determine which router will be the active router.
	- E.g., `standby 1 priority 200`
- `standby GROUP_NUMBER preempt`
	- Causes the router to take the role of active router, even if another router already has the role.
	- However, the pre-empting router must have a higher priority or IP address.
	- Only necessary on the router you want to become active.
## Standard ACL Configuration
### ACL Show Commands
- `show access-lists` Display all kinds of ACLs.
	- Shows how many pings were blocked. ![[Pasted image 20240903161652.png]]
- `show ip access-lists` Display the IP ACLs.
- `show running-config | include access-list` Show lines in the commands that include "access-list". (Remark is displayed)
	![[Pasted image 20240903140738.png]]
	- Each entry is given a number indicating the order. The default entry number will be 10, 20, 30, etc. (Yellow box)
### ACL Apply Command
- `ip access-group NUMBER {in | out}`
	- Configured in interface mode: apply the ACL to an interface.
	- Remember ACLs have to be applied on interfaces after creation!!
### ACL Edit Command
- `ip access-list resequence ACL_ID STARTING_SEQ_NUM INCREMENT`
	- Works in global config mode.
	- Re-sequencing function that helps edit ACLs
	- E.g., Entries were sequenced 1, 2, 3, 4, and 5. This is not ideal because it is impossible to insert new entries in between.
	- `ip access-list resequence 1 10 10` ![[Pasted image 20240903180253.png]]
		- `1`: ACL number
		- First `10`: Change the sequence number of the first entry to 10. 
		- Second `10`: Add 10 for every entry after that.
### ACL Delete Command
- `no ENTRY_NUMBER`
	![[Pasted image 20240903174810.png]]
	- A way to delete individual entries in the ACL.
	- Adding `no` in front of the commands will not work as usual, as it will delete the entire ACL! ![[Pasted image 20240903175104.png]]
### Standard Numbered ACL Configuration
Configured directly in global config mode:
- `access-list ACL_NUMBER {deny | permit} IP WILDCARD_MASK`
	- Basic command to configure a standard numbered ACL.
	- Wildcard mask is not required if you specify a /32 mask. 
	- E.g., The following three commands deny a single host `1.1.1.1/32`. ![[Pasted image 20240903135710.png]]
- `access-list 1 permit any`
	- Used to add the entry to permit traffic. ![[Pasted image 20240903140053.png]]
- `access-list ACL_NUMBER remark REMARK`
	- Adds a comment that helps you remember the purpose of the ACL.
	- E.g., `access-list 1 remark ## BLOCK BOB FROM ACCOUTING ##`
- Example Configuration:
	![[Pasted image 20240903144401.png]]
	![[Pasted image 20240903144424.png]]
	- Rule-of-thumb: Standard ACLs should be applied **as close to the destination as possible**.
	- In the example below, the ACL should only control traffic that tries to access the `192.168.2.0/24` network.
		![[Pasted image 20240903142343.png]]
		![[Pasted image 20240903142919.png]]
	- Therefore the ACE should be applied outbound on G0/2 of R1.
- `ip access-list standard NUMBER`
	- Another way to configure numbered ACLs, but in the form of named ACLs. ![[Pasted image 20240903174532.png]]
	- Easier to delete individual entries with `no ENTRY_NUMBER`
	- WARNING - When configuring/ editing numbered ACLs from global config mode, **you can’t delete individual entries**, you can only delete entire ACL!
### Standard Named ACL Configuration
Configured with subcommands in a separate config mode:
- `ip access-list standard ACL_NAME`
	- Enter standard name config mode.
	- E.g., `ip access-list standard BLOCK_BOB`
- `[ENTRY-NUMBER] {deny | permit} IP WILDCARD_MASK`
	- Configure the deny and permit entries.
	- Entry numbers can be specified to control the order.
		- E.g., A new entry with sequence number 30 was added, which goes between entries 20 and 40.  ![[Pasted image 20240903175723.png]]
	- E.g., `5 deny 1.1.1.1`, `10 permit any`
- Example Configuration: 
	![[Pasted image 20240903144401.png]]
	![[Pasted image 20240903144312.png]]
	- 2 ACLs are used to follow the requirements at different interfaces, to control access to 2 servers.
	![[Pasted image 20240903144617.png]]![[Pasted image 20240903144725.png]]
	- The router may re-order the /32 entries that match a single specific host to improve efficiency of processing the ACL. (Does not change the overall effect of the ACL)
## Extended ACL Configuration
### Extended Numbered ACL Configuration
- `access-list NUMBER [permit | deny] PROTOCOL SRC_IP DEST_IP`
	- Protocol: TCP, UDP, etc.
	- Can be configured on global config mode using the extended named configuration.
### Extended Named ACL Configuration
- `ip access-list extended {NAME | NUMBER}`
	- Extended numbered ACL can also be configured in this mode.
- `[SEQ_NUM] [permit | deny] PROTOCOL SRC_IP DEST_IP`