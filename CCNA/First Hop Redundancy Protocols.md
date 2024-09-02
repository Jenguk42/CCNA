FHRP is a computer networking protocol which is designed to protect the default gateway used on a subnetwork by allowing two or more routers to provide backup for their address; in the event of failure of an active router, the backup router will take over the address, usually within a few seconds.
- Default gateway is the “first hop” - first router in the path to whatever destination the PC is sending traffic to.
- Troubleshooting tip: Make sure the versions match, and the terminologies/Multicast IP/Victual MAC are configured properly!
## Purpose of FHRPs
Redundant connections help with keeping network connectivity.
- E.g., If internet connection from R1 fails, PC1 can connect through R2.
	- This may be a slower and cheaper connection until R1’s connection is recovered. 
![[Pasted image 20240829195718.png]]
 - However, the PCs do not know that R1 is down, and still have their default gateway set as R1.
 - How do we cause R2 to automatically become the default gateway? ➡️ via **First Hop Redundancy Protocols**
## How does FHRP Work?
Each FHRP uses different terms and formats.
- A **virtual IP** is configured on the two routers, and a **virtual MAC** is generated for the virtual IP. 
	- If there are multiple subnets & VLANs, IP addresses should be separately configured because each subnet needs its own default gateway.
- An **active** router and a **standby** router are elected.
	![[Pasted image 20240829204459.png]] ![[Pasted image 20240829204649.png]]
	- The role is negotiated by exchanging multicast Hello messages.
	- Active router is determined in this order:
		1. Highest priority (default 100, can be configured)
		2. Highest IP address
	- R1: Active, will act as the default gateway.
	- R2: Standby, will not function as the default gateway unless the active fails.
- End hosts in the network are configured to use the virtual IP as their default gateway.
	![[Pasted image 20240829204850.png]]
	![[Pasted image 20240829204952.png]]
- The active router replies to ARP requests using the virtual MAC address, so traffic destined for other networks will be sent to it. (Functions as the default gateway for the network.)
	![[Pasted image 20240829205107.png]]
	PC1 will connect to the internet like this: ![[Pasted image 20240829205217.png]]
- If the active router fails, the standby becomes the next active router.
	![[Pasted image 20240829205252.png]]
- The new active router will send **gratuitous ARP** messages so that switches will update their MAC address tables. It now functions as the default gateway. ![[Pasted image 20240829205645.png]]
	- Gratuitous ARP: ARP replies sent without being requested (no ARP request messages was received.)
	- The frames are broadcast to `FFFF.FFFF.FFFF`. (Normal ARP replies are unicast!)
- Because the messages are broadcast, all switches receive it and update their MAC address tables. The PC’s ARP tables are not updated.
	![[Pasted image 20240829205925.png]]
	PCs will connect to the internet through R2, like this:
	![[Pasted image 20240829210022.png]]
- If the old active router comes back online, by default it won’t take back its role as the active router. It will become the standby router.
	- The current active router will not automatically give up its role because FHRPs are ‘non-pre-emptive’.
	- You can configure ‘pre-emption’, so that the old active router does take back its old role.

Summary:
1. An Active and a Standby router are set by Hello messages. 
2. The two routers share a virtual IP addresses and a MAC addresses. Active router takes the frames forwarded by the PCs.
3. If the Active router fails, the Standby router becomes the active router and asks the switches to update their MAC address tables through Gratuitous ARP messages.
4. FHRPs are ‘non-pre-emptive’, so even if the former active router returns, the current active router will not automatically give up its role. 
## Three Major FHRPs
![[Pasted image 20240829212338.png]]
Functionality of each is very similar!
### 1. HSRP (Hot Standby Router Protocol)
- Cisco proprietary, can only be run on Cisco
- **Active** and **Standby** router are elected.
- Refer to [[Router CLI Commands#HSRP Configuration]] for configuration.
- Two versions, V1 and V2:
	- Version 2 adds IPv6 support and increases the number of groups that can be configured.
	- Versions 1 and 2 are not compatible, so all routers using the protocol should be using the same version.
- Multicast IPv4 addresses:
	- V1 = `224.0.0.2`
	- V2 = `224.0.0.102`
- Virtual MAC addresses:
	- V1 = `0.0.0.0.0c07.acXX` (`XX` = HSRP group number)
	- V2 = `0.0.0.0.0c9f.fXXX` (`XXX` = HSRP group number)
- In a situation with multiple subnets/VLANs, you can configure a different active router in each subnet/VLAN to load balance.
	![[Pasted image 20240829210913.jpg]]
	- E.g., R1 can be active and R2 can be standby in VLAN1, while R2 can be active and R1 can be standby in VLAN2.
### 2. VRRP (Virtual Router Redundancy Protocol)
- Open standard, can be run on any router (including Cisco).
- Nearly identical to HSRP, except a few differences.
	- A **master** and **backup** router are elected.
		- Functions are the same to active and standby.
	- Multicast IPv4 address: `224.0.0.18`
	- Virtual MAC addresses: `0000.5e00.01XX` (`XX` = VRRP group number)
- In a situation with multiple subnets/VLANs, you can configure a different master router in each subnet/VLAN to load balance.
### 3. GLBP (Gateway Load Balancing Protocol)
- Different from HSPR and VRRP!
- Cisco proprietary
- IMPORTANT: Load balances among multiple routers within a single subnet.
	- E.g., If PC1 and PC2 are both in VLAN1, PC1 can use R1 as its default gateway, and PC2 can use R2 as its default gateway.
- A single **AVG (Active Virtual Gateway Forwarder)** is selected.
- Up to four **AVFs (Active Virtual Forwarders)** are assigned by the AVG.
	- The AVG itself can be an AVF.
	- Each AVF acts as the default gateway for a portion of the hosts in the subnet.
- Multicast IPv4 address: `224.0.0.102`
- Virtual MAC address: `0007.b400.XXYY`
	- `XX` = GLBP group number
	- `YY` = AVF number
