ACLs (Standard Access Control Lists) have multiple uses, but this note focuses on ACLs from a security perspective (E.g., Controlling which devices have access to which parts of the network).
- ACLs function as a **packet filter**, instructing the router to permit or discard specific traffic.
	- Even if the router has a route to the destination, the packet may be discarded if the ACL tells the router to do so.
- ACLs filter traffic based on:
	- Source/destination **IP addresses**,
	- Source/destination **Layer 4 ports**,
	- etc.
## How do ACLs Work?
ACLs are configured globally on the router (global config mode). They are an ordered sequence of ACEs (Access Control Entries). 
### Basic Rules with Example network Topology 
![[Pasted image 20240903131116.png]]
- REQUIREMENTS:
	- Hosts in `192.168.1.0/24` can access the `10.0.1.0/24` network. (PC1 and 2 should have access to SRV1.)
	- Hosts in `192.168.2.0/24` cannot access the `10.0.1.0/24` network. (PC3 and 4 should not have access to SRV1.)
- ACL1:
	1. If source IP = `192.168.1.0/24`, then permit.
	2. If source IP = `192.168.2.0/24`, then deny.
	3. If source IP = any, then permit.

1. Configuring an ACL in global config mode will not make the ACL take affect; after being created, the ACL must be **applied** to an interface.
	- When the router checks a packet against the ACL, it processes the ACEs in order, from top to bottom.
	- If the packet matches one of the ACEs in the ACL, the router takes action and stops processing the ACL. All entries below the matching entry will be ignored.
2. ACLs can be **inbound** or **outbound**.
	- Rule-of-thumb: Standard ACLs should be applied **as close to the destination as possible**.
	- Configuring the correct **interfaces** in the correct **direction** is crucial to succeed in meeting the requirements.
	- E.g., If ACLs are applied outbound on R1's G0/2, it will only take effect on traffic exiting G0/2.
		- Traffic entering G0/2 from PC3 will not be filtered, and will successfully reach SRV1.
		- When SRV1 sends a reply to PC3, all ACLs are not met. (The source IP is `10.0.1.100`.)
		- This would not fulfil the requirements.
	- E.g., If ACLs are applied inbound on R1's G0/2, it will only take effect on traffic entering G0/2.
		- If PC3 tries to send traffic to SRV1, ACE 2 is met.
		- The traffic is denied, which fulfils the requirement?
		- Not really - this blocks hosts in `192.168.2.0/24` from communicating with all other networks outside of their local LAN!
	- Best option: outbound on R2's G0/1 interface.
3. A maximum of one ACL can be applied to a single interface *per direction*.
	- Inbound: Max one ACL, Outbound: Max one ACL
	- If you apply multiple ACL to the same interface in the same direction, it will replace the previous one.
1. There is an **implicit deny** at the end of all ACLs.
	- If a packet doesn't match any of the configured entries in an ACL, the router will **deny** the packet.
	- This is as if there is an invisible entry at the end, "if source IP = any, then deny".
	![[Pasted image 20240903134502.png]]
## Standard ACLs
Match traffic based only on the **source IP addresses** of the packet. Multiple ACLs can be configured on a single router, so you need to identify each ACL.
### Standard Numbered ACLs
Identified with a number. (e.g., ACL 1, ACL 2, etc.)
- Different types of ACLs have a different range of numbers that can be used: Standard ACLs can use **1-99 and 1300-1999**.
- Less specific, should be applied as close to the **destination** as possible.
	- If applied close to the source, there is a risk of blocking more traffic than intended.
- Basic command to configure a standard numbered ACL: `access-list NUMBER [deny | permit] IP WILDCARD_MASK`.
- Remember to add the entry to permit traffic: `access-list NUMBER permit any`.
- Refer to [[Router CLI Commands#Standard Numbered ACL Configuration]] for Router CLI commands.
### Standard Named ACLs
Identified with a name. (E.g., 'BLOCK_BOB')
- Configured by entering 'standard named ACL config mode', then configuring each entry within that config mode.
- To enter the config mode, use `ip access-list standard ACL_NAME`
Refer to [[Router CLI Commands#Standard Named ACL Configuration]] for Router CLI commands.
### Advantages of Named ACL Config Mode
- You can easily delete individual entries in the ACL with `no SEQUENCE_NUMBER`. (Refer to [[Router CLI Commands#ACL Delete Command]])
- You can insert new entries in between other entries by specifying the sequence number.
## Extended ACLs
Match traffic based on Source/Destination IP, Source/Destination port, etc.
- More parameters - more precise and complex (than standard ACLs)
	- You can permit or deny specific kinds of traffic from specific hosts to specific destinations.
	- Therefore should be applied as close to the **source** as possible, to limit how far the packets travel in the network before being denied.
		- Less risk of blocking more traffic than intended.
		- More efficient - routers will not waste resources processing packets that will just be dropped.
- Can be numbered or named, just like standard ACLs.
	- Numbered ACLs use the range of **100 - 199** and **2000 - 2699**.
- Basic command to configure an extended standard ACL: `access-list NUMBER [permit | deny] PROTOCOL SRC_IP DEST_IP`
	- Main parameters: **Layer 4 protocol/port**, **source address**, and **destination address**.
	- Refer to [[Router CLI Commands#Extended ACL Configuration]] for specific configuration.
- Matching the Layer 4 protocol
	![[Pasted image 20240903181555.png]]
	- IP protocol number: Protocol field in the IPv4 header. (E.g., TCP, UDP, etc.)
		- 1: ICMP
		- 6: TCP
		- 17: IDP
		- 88: EIGRP
		- 89: OSPF
	- You can also just use the names.
	- `ip`: Matches all IP packets; used when we don’t care about the protocol.
		- E.g., When adding `permit any` statement
- Source & Destination IP Addresses  ![[Pasted image 20240903182257.png]]
	- Note that the `host` option should be used when specifying a /32 source or destination, or specify the wildcard mask.
	- E.g., `deny tcp any 10.0.0.0 0.0.0.255`
		- Denies all packets that encapsulate a TCP segment, from any source IP address to destination network `10.0.0.0/24`.
- Matching TCP/UDP port numbers (Optional)
	- When matching TCP/UDP, you can optionally specify the source and/or destination port numbers to match.
	- `[permit | deny] PROTOCOL SRC_IP {eq | gt | lt | neq | range} SRC_PORT_NUM DEST_IP {eq | gt | lt | neq | range} DEST_PORT_NUM`
		- Source IP with the source port matches then destination IP with the destination port matches.
		- `eq 80` = equal to port 80, matches TCP source port 80 (The most common!)
		- `gt 80` = greater than port 80 (81 and greater)
		- `lt 80` = less than 80 (79 and less)
		- `neq 80` = NOT 80
		- `range 80 100` = From port 80 to port 100
	- Port numbers: ![[Pasted image 20240903184140.png]]
	- Try to remember these: ![[Pasted image 20240903184039.png]]

- Extended ACL Entry Practice (1)
	- Allow all traffic:
		- `permit ip any any`
	- Prevent `10.0.0.0/16` from sending UDP traffic to `192.168.1.1/32`:
		- `deny udp 10.0.0.0 0.0.255.255 192.168.1.1 0.0.0.0`
		- = `deny udp 10.0.0.0 0.0.255.255 host 192.168.1.1`
	- Prevent `172.16.1.1/32` from pinging hosts in `192.168.0.0/24:`
		- `deny icmp 172.16.1.1 0.0.0.0 192.168.0.0 0.0.0.255`
		- = `deny icmp host 172.16.1.1 192.168.0.0 0.0.0.255`

- Extended ACL Entry Practice (2)
	- Allow traffic from `10.0.0.0/16` to access the server at `2.2.2.2/32` using HTTPS.
		- `permit tcp 10.0.0.0 0.0.255.255 host 2.2.2.2 eq 433`
		- HTTPS uses tcp, and its port number is `433`.
	- Prevent all hosts using source UDP port numbers from `20000` to `30000` from accessing the server at `3.3.3.3/32`.
		- `deny udp any range 20000 30000 host 3.3.3.3`
	- Allow hosts in `172.16.1.0/24` using a TCP source port greater than `9999` to access all TCP ports on server `4.4.4.4/32` except port 23.
		- `permit tcp 172.16.1.0 0.0.0.255 gt 9999 host 4.4.4.4 neq 23`
## Extended ACL Example Configuration
![[Pasted image 20240903144401.png]]
![[Pasted image 20240903203044.png]]
- Extended ACL entries:
	- Requirement 1 = `deny tcp 192.168.1.0 0.0.0.255 host 10.0.1.100 eq 443`
	- Requirement 2 = `deny ip 192.168.2.0 0.0.0.255 10.0.2.0 0.0.0.255`
	- Requirement 3 requires three deny entries:
		- `deny icmp 192.168.1.0 0.0.0.255 10.0.1.0 0.0.0.255`
		- `deny icmp 192.168.1.0 0.0.0.255 10.0.2.0 0.0.0.255`
		- `deny icmp 192.168.2.0 0.0.0.255 10.0.1.0 0.0.0.255`
		- From requirement 2, all traffic from `192.168.2.0/24` to `10.0.2.0/24` was already blocked. So we don’t need to block again. 
- Create ACLs, for all requirements:
	- Enter extended name configuration mode, and give the ACLs a name = `ip access-list extended HTTP_SRV1`
	- Enter the ACL entries.
	- Remember to use `permit ip any any` to allow all other traffic.
- Apply ACLs on the interfaces, as close to the source as possible.
	- Requirement 1 should be applied inbound on R1’s G0/1 interface = `ip access-group HTTP_SRV1 in` on R1’s G0/1. ![[Pasted image 20240903204931.png]]
	- Requirement 2 should be applied inbound on R1’s G0/2 interface = `ip access-group BLOCK_10.0.2.0/24 in` on R1’s G0/2.
	- Requirement 3 should be applied outbound on R1’s G0/0 interface = `ip access-group BLOCK_ICMP out` on R1’s G0/0.
- Final Check: 
	![[Pasted image 20240903205739.png]]
	![[Pasted image 20240903205809.png]]