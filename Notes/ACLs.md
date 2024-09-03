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
- Basic command to configure a standard numbered ACL: `access-list NUMBER {deny | permit} IP WILDCARD_MASK`.
- Remember to add the entry to permit traffic: `access-list NUMBER permit any`.
- Refer to [Standard Numbered ACL configuration](<Router CLI Commands#Standard Numbered ACLs>) for Router CLI commands.
### Standard Named ACLs
Identified with a name. (E.g., 'BLOCK_BOB')
- Configured by entering 'standard named ACL config mode', then configuring each entry within that config mode.
Refer to [Standard Named ACL configuration](<Router CLI Commands#Standard Named ACLs>) for Router CLI commands.
## Extended ACLs
Match based on Source/Destination IP, Source/Destination port, etc.
### Extended Numbered ACLs
### Extended Named ACLs