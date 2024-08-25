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
### Static Route Configuration
Refer to [[Static Routing]] for detailed explanation
![[Pasted image 20240814153912.png]]
- `ip route IP-ADDRESS NETMASK NEXT-HOP` (Preferred)
	- Configures the next-hop to reach the IP address
	- E.g., For R1 to reach PC4, it should hop to `G0/0` interface of R3 which has an IP address of `192.168.13.3`.
		- `ip route 192.168.4.0 255.255.255.0 192.168.13.3`

- `ip route IP-ADDRESS NETMASK EXIT-INTERFACE`
	- Configures which interface to send the packet out of to reach the IP address
	- E.g., For R2 to reach PC1, it should send the packet out of the `G0/0` interface.
		- `ip route 192.168.1.0 255.255.255.0 g0/0`
	- Shown in the routing table as "directly connected", although it is not exactly directly connected
		- ![[Pasted image 20240815133432.png]]
		- If you only specify the exit-interface, the router will have to use ARP (Proxy ARP) for every destination it wants to reach, so it can be considered worse.
		- Stick to using the next-hop in your static routes!

- `ip route IP-ADDRESS NETMASK EXIT-INTERFACE NEXT-HOP`
		- Configures both next-hop and interface
	- E.g., For R2 to reach PC4, it should:
		- Send the packet out of the `G0/1` interface and 
		- Hop to `G0/0` interface of R4 which has an IP address of `192.168.24.4`.
		- `ip route 192.168.4.0 255.255.255.0 g0/1 192.168.24.4`

- `ip route 0.0.0.0 0.0.0.0 DEFAULT-ROUTE
	![[Pasted image 20240815134608.png]]
	- Configures the  default route
	- `*` means that this route is the candidate to become the router's default route.
	- Possible to have multiple candidates
### Floating Static Route Configuration
- `ip route DESTINATION_IP SUBNET_MASK AD`
	![[Pasted image 20240822163957.png]]
	- Static route is configures as usual, but with a different administrative distance.
	- If you want to keep the dynamic route as default and floating static route as a backup, AD should be higher than the dynamic route.
		- E.g., OSPF has an AD of 110, so set the floating static route with AD 111.
		- In this case, the static route will not show up in the routing table because the OSPF route is favoured and written on the table.
### Dynamic Routing Configuration
### Commands used in Common
- `show ip protocols`: Shows various stats
	- Example: RIP ![[Pasted image 20240823143643.png]]
- `maximum paths NUMBER`: Change the maximum paths that can be saved for the same destination. NEEDS TO BE EXECUTED IN PROTOCOL CONFIGURATION MODE
	![[Pasted image 20240823143929.png]]
- `distance NUMBER`: Change the administrative distance to make it favoured over other protocols. NEEDS TO BE EXECUTED IN PROTOCOL CONFIGURATION MODE ![[Pasted image 20240823144145.png]]
- `network IP_ADDRESS`: "Activate IGP on interfaces with an IP address that falls under the given IP range." 
	![[Pasted image 20240823134517.png]]
	- Used in RIP, OSPF, and EIGRP
	- Automatically converts the IP address to classful networks.
		- E.g., command `network 10.0.12.0` will be converted to `network 10.0.0.0/8` (Class A network).
		- No need to enter the network mask
	- Tells the router to:
		- Look for interfaces with an IP address that is in the specified range.
		- Activate RIP on the interfaces that fall in the range.
		- Form adjacencies with connected RIP neighbours.
		- Advertise **the network prefix of the interface** (NOT the prefix in the `network` command)
	- This command doesn't tell the router which networks to advertise, it tells the router **which interfaces to activate RIP on.**
		- The router will advertise the network prefix of these interfaces.
	
- `passive-interface INTERFACE_ID`
	- **DONE IN RIP CONFIGURATION MODE, NOT ON THE INTERFACE!**
		- This is why the interface should be specified in the command.
	- Tells the router to stop sending RIP advertisements out of the specified interface.
	- The router will still advertise the network prefix of the interface to its RIP neighbours.
	- Should be used on interfaces which don't have any RIP neighbours. 
	- Example scenario:
		- In the case below, R1 will continuously send RIP advertisements out of G2/0 although there is no RIP neighbours connected. 
		- This is unnecessary traffic, so G2/0 should be configured as **passive interface**. 
		![[Pasted image 20240823141421.png]]

- `default-information originate`
	- Share the default route with the neighbouring same-protocol-enabled routers.
	- Example Scenario:
		![[Pasted image 20240823143041.png]]
		- R1 configured the default route with the command: `ip route 0.0.0.0 0.0.0.0 203.0.113.2`.
		- After sharing the default route with `default-information originate`, R2, R3, and R4 learns the route too.
			![[Pasted image 20240823143230.png]]
			- This is an example of RIP enabled routers. Both routes (To R3, `10.0.34.1` via interface `F2/0`; To R2, `10.0.24.1` via interface `G0/0`) are stated since they have the same hop count. 
			- R4 will load-balance traffic over the two routes.
#### RIP Configuration
![[Pasted image 20240823132552.png]]
- `router rip`: Enter RIP configuration mode
- `version 2`: Use RIP version 2 to allow VLSM and CIDR 
- `no auto-summary`: Use classless networks
	- `auto-summary` is on by default, and it automatically converts the networks (advertised by the router) to classful networks.
	- E.g., `172.16.1.0/28` attached to R1 is converted to class B network, advertised as `172.16.0.0/16`
#### EIGRP Configuration
![[Pasted image 20240823145628.png]]
- `router eigrp AS`
	- AS (Autonomous System) number must match between routers, or they will not form an adjacency and share route information.
- `no auto-summary`: Use classless networks.
- A **wildcard mask** can be used with the `network` command!
	- It will assume a classful address if you don't specify the mask. (E.g., `10.0.0.0` is assumed to be `10.0.0.0/8`)
	- Usually the wildcard mask will have the same prefix length as the interface itself.
- `eigrp router-id ROUTER_ID`
	![[Pasted image 20240823153041.png]]
	- Manual configuration of router ID (Highest priority)
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
