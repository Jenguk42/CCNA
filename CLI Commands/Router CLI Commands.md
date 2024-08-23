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
### Dynamic Route Configuration
- `ip route DESTINATION_IP SUBNET_MASK AD`
	![[Pasted image 20240822163957.png]]
	- Used to change the administrative distance.
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
