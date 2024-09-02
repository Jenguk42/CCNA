Starting to take over networks all over the world! Offers a lot of improvement.
- **Dual Stack**: A way of transitioning the network from IPv4 to IPv6 - You keep IPv4 running, add IPv6 on top, and slowly transition to using IPv6 exclusively.
## What happened to IPv5?
- Internet Stream Protocol was developed in the late 1970s, but never actually introduced for public use.
- It was never called IPv5, but it used a value of 5 in the Version field of the IP header.
- So, when the successor to IPv4 was being developed, it was named IPv6 to avoid Confusion.
## Why IPv6?
Main reason: There aren’t enough IPv4 addresses available! We need a **long-term solution**.
- There are 2^32 (4,294,967,296) IPv4 addresses available.
- The creators who designed IPv4 had no idea this would be enough.
- VLSM,  private IPv4 addresses, and NAT have been used to conserve the use of IPv4 address space; but these are short-term solutions.

IPv4 address assignments are controlled by IANA (Internet Assigned Numbers Authority)
- IANA distributes IPv4 address space to various RIRs (Regional Internet Registries), which then assign them to companies that need them...
- … Until they ran out. ![[Pasted image 20240901141510.png]]
## What is IPv6?
- An IPv6 address is 128 bits.
	- 4 times the bits of an IPv4 address != 4 times the number of possible addresses;
	- Every additional bit doubles the number of possible addresses.
	- There are 340,283,366,920,938,463,374,607,431,768,211,465 IPv4 addresses!
- IPv6 addresses are written as 32 hexadecimal characters, divided into 8 groups of 4 using colons.
	- Each hexadecimal digits contain 4 bits of information (Refer to [[Hexadecimal]]), 128 / 4 = 32
	- Each digit of IPv4 address contains 1 bit of information, so IPv6 addresses contain 4 times the info.  
	- E.g., `2001:0db8:5917:eabd:6562:17ea:c92d:59bd`
- Uses the ‘slash’ notation to indicate prefix length. (No more subnet masks)
	- E.g., `/64`: The first half is the network portion, and the second half is the host portion.
- RFC 5952: 'A Recommendation for IPv6 Address Text Representation'
	- RFC (Request for Comments) is a publication from the ISOC (Internet Society) and associated organisations like the IETF (Internet Engineering Task Force), and are the official documents of Internet specifications, protocols, procedures, etc.
	- Suggests standardising IPv6 address representation:
		- Leading 0s MUST be removed.
		- `::` MUST be used to shorten the longest string of all-0 quartets.
		- If there is only one all-0 quartet, don't use `::`.
		- If there are two equal-length choices for the `::`, shorten the one on the left.
		- Hexadecimal characters `a`, `b`, `c`, `d`, `e`, and `f` MUST be written using lower-case, NOT upper-case.
	- E.g., `2001:0000:0000:0000:0f2a:0000:0000:00b1` -> `2001::f2a:0:0:b1`
## IPv6 Address Abbreviation
### Shortening (Abbreviating) IPv6 Addresses
- **Leading** 0s can be removed. ![[Pasted image 20240901142553.png]]
	- Only the LEADING zeros!
- Consecutive quartets of all 0s can be replaced with a double colon (`::`). ![[Pasted image 20240901142636.png]]
	- This is possible because we know that IPv6 addresses are 8 quartets in length.
	- However, **consecutive quartets of 0s can only be abbreviated once in an IPv6 address.** ![[Pasted image 20240901142902.png]]
		- The second and third quartets of 0 cannot be removed, you can only remove the leading 0s.
- Both methods can be combined. ![[Pasted image 20240901142803.png]]
- Practice questions:
	- `2000:Ab78:0020:01bf:ed89:0000:0000:0001`
		-> `2000:Ab78:20:1bf:ed89::1`
	- `fe80:0000:0000:0000:0002:0000:0000:fbe8`
		-> `fe80::2:0:0:fbe8`
	- `ae89:2100:01ac:00f0:0000:0000:0000:020f`
		-> `ae89:2100:1ac:f0::20f
	- `2001:0db8:8b00:1000:0002:0bc0:0d07:0099`
		-> `2001:db8:8b00:1000:2:bc0:d07:99`
### Expanding Shortened IPv6 Addresses
1. Put leading 0s where needed (All quartets should have 4 hexadecimal characters).
	![[Pasted image 20240901143539.png]]
2. If a double colon is used, replace it with all-0 quartets. Make sure there are 8 quartets in total.
	![[Pasted image 20240901143623.png]]
- Practice questions:
	- `fe80::1010:2fc:0:9`
		-> `fe80:0000:0000:0000:1010:02fc:0000:0009`
	- `ff02::2`
		-> `ff02:0000:0000:0000:0000:0000:0000:0002`
## Finding the IPv6 Prefix (Global Unicast Address Type)
![[Pasted image 20240901144541.png]]
Typically, an enterprise requesting IPv6 addresses from their ISP will receive a /48 block, and IPv6 subnets use a /64 prefix length.
- This means an enterprise has **16 bits to use to make subnets.**
- The remaining 64 bits can be used for hosts.
### Prefix length is 64
Make the second half of the address all 0s. First half remains, and the second half is replaced with a double colon.
### Prefix length is a multiple of 4
Check up to which digit is counted as the network portion.
- Easier to count because each digit contains 4 bits of information: ![[Pasted image 20240901150018.png]]
- Everything after is the host portion.
### Prefix length is not a multiple of 4
 This means that the network portion ends in the middle of a hexadecimal digit.  ![[Pasted image 20240901150740.png]]
 - The network portion includes everything before the `b` and *the first bit* of the `b`.
 - Look into the binary: `0xb` = `0d11` = `0b1011`, and  only the first bit is part of the network portion of the address.
 - Change all of the other bits: `0b1000` = `0d8` = `0x8`
 - In the network prefix, `b` should be changed to `8`.
### Practice Questions
![[Pasted image 20240901150945.png]]
## IPv6 Address Types
### Global Unicast Addresses
![[Pasted image 20240901144541.png]]
Public addresses which can be used over the internet.
- Must register to use them. Because they are public addresses, they are expected to be globally unique.
	- Two companies using the same global unicast address is going to cause a problem. (It's like two houses having the same street address.)
- Originally defined as the `2000::/3` block.
	- Includes all addresses from `2000::` to `3fff:ffff:ffff:ffff:ffff:ffff:ffff:ffff`.
- Now defined as all addresses which aren't reserved for other purposes.
Comes in three parts:
- Network Prefix
	- **Global Routing Prefix**: Assigned to the company by the ISP. The company is free to use this /48 block of addresses however they want.
	- **Subnet Identifier**: Remaining 16 bits of the prefix, used to make various subnets. (Allows over 65,000 subnets.)
- **Interface Identifier**: Host portion, can be generated with EUI-64 or manually configured.
### Unique Local Addresses
![[Pasted image 20240902110022.png]]
Private addresses which cannot be used over the internet.
- Do not need to register to use them. Can be used freely within internal networks and don't need to be globally unique.
	- The global ID should be unique, so that addresses don't overlap when companies merge.
- Cannot be routed over the internet.
	- ISP will drop packets destined for unique local addresses.
- Originally defined as the `fc00::/7` block (from `fc00::` to `fdff:ffff:ffff:ffff:ffff:ffff:ffff:ffff`).
	- A later update requires the 8th bit to be 1, so the **first two digits must be `FD`.** 
Comes in four parts:
- Network Identifier:
	- `fd`: Indicates that the address is a unique local address.
	- **Global ID**: *Recommended* to be randomly generated.
	- **Subnet Identifier**: Remaining 16 bits of the prefix, used to make various subnets. (Allows over 65,000 subnets.)
- **Interface Identifier**: Host portion, can be generated with EUI-64 or manually configured.
### Link Local Addresses
Automatically generated on IPv6-enabled interfaces.
- Use command `ipv6 enable` on an interface to enable IPv6 on an interface.
	- If an IPv6 address is not manually provided, the link local address will be the only IPv6 address on the interface.
- Defined as the `fe80::/10` block (from `fe80::` to `febf:ffff:ffff:ffff:ffff:ffff:ffff:ffff`).
	- However, the standard states that the 54 bits after `fe80/10` should be all 0.
	- Therefore the link local addresses always begin with `fe8`.
- Interface ID is generated using EUI-64 rules.
*Link-local* means that these addresses are used for communication within a single link (subnet).
- Routers **will not** route packets with a link-local destination IPv6 address. 
- = They will not forward packet between subnets. 
Common uses:
- Routing protocol peerings (OSPFv3 uses link-local addresses for neighbour adjacencies, sending LSAs, etc.).
- Next-hop address for static routes. ![[Pasted image 20240902111115.png]]
- Neighbour Discovery Protocol (NDP, IPv6's replacement for ARP) uses link-local addresses to function.
### Multicast Addresses 
- **Unicast**: addresses are "one-to-one", one source to one destination.
	![[Pasted image 20240902145027.png]]
- **Broadcast**: addresses are "one-to-all", one source to all destinations (within the subnet).
	![[Pasted image 20240902145118.png]]
- **==Multicast==**: addresses are "one-to-many-of-many", one source to multiple destinations (that have joined the specific *multicast group*).
	![[Pasted image 20240902144941.png]]
**IPv6 doesn't use broadcast**, but uses the all **nodes IPv6 address** (`ff02::1`) to send a message to all hosts in the subnet. It functions like a broadcast since it's destined for all hosts.
![[Pasted image 20240902111553.png]]
- All last digits are the same, for IPv4 and IPv6.
- Range `ff00::/8` is used (`ff00::` to `ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff`).
#### Solicited-Node Multicast Address
An IPv6 solicited-node multicast address is calculated from a unicast address.
![[Pasted image 20240902153649.png]]
- `ff02:0000:0000:0000:0000:0001:ff` + Last 6 hex digits of unicast address

![[Pasted image 20240902144101.png]]
The `joined group address(es)` shows the groups that R1 has joined in its interface:
- `ff02::1` All nodes multicast address. R1's G0/0 interface is a host in the `2001:db8::/64` subnet, so it automatically joins this group. 
- `ff02::2` All routers multicast address. R1 is a router, so it automatically joins this group.
- `ff02::1:ff36:8500` Solicited-node multicast address, prefix (`FF02::1:FF`) + the last 6 digits of the global unicast address (`36:8500`).
#### Multicast Address Scopes
Demonstration of how each scope can reach hosts within & beyond the organisation:
![[Pasted image 20240902143422.png]]
IPv6 defines multiple multicast 'scopes' which indicate how far the packet should be forwarded.
- **Interface-local** (`ff01`): The packet doesn't leave the local device. Can be used to send traffic to a service within the local device.
- **Link-local** (`ff02`): The packet remains within the local subnet. Routers will not route the packet between subnets.
- **Site-local** (`ff05`): The packet can be forwarded by routers. Should be limited to a single physical location (not forwarded over a WAN).
- **Organisation-local** (`ff08`): Wider in scope than site-local (an entire company/ organisation).
- **Global** (`ff0e`): No boundaries. Possible to be routed over the internet.
The actual boundary of site-local, organisation-local, and global scope should be defined by the network engineer.
### Anycast Addresses
"One to one of many"
![[Pasted image 20240902144903.png]]
- New feature of IPv6, traffic is sent to one of the multiple possible destinations.
- Multiple routers are configured with the same IPv6 address.
	- They use a routing protocol to advertise the address.
	- When hosts send packets to the destination address, routers forward it to the nearest router configured with that IP address (based on the routing metric).
- There is no specific address range - use a regular unicast address (global unicast, unique local) and specify it as an anycast address:
	- E.g., `ipv6 address 2001:db8:1:1::99/128 anycast` on an interface ![[Pasted image 20240902145542.png]]
### Other IPv6 Addresses
- `::` = The unspecified IPv6 address
	- Can be used when a device doesn't yet know its IPv6 address.
	- IPv6 default routes are configured to `::/0`.
	- IPv4 equivalent: `0.0.0.0`
- `::1` = The loopback addresses
	- Used to test the protocol stack on the local device.
	- Messages sent to this addresses are processed within the local device, but not sent to other devices.
	- IPv4 equivalent: `127.0.0.0/8` address range
## IPv6 Address Configuration
R1 below has 3 interfaces, each connected to a different subnet. The company was assigned a /64 address block, and is using a last quartet of its prefix to make different subnets.
![[Pasted image 20240901151059.png]]
![[Pasted image 20240901151220.png]]
`ipv6 unicast-routing`
- Allows the routers to perform IPv6 routing
- If this is not done, end hosts cannot reach hosts connected to the router's other interface.
`ipv6 address IPv6_ADDRESS`
- IPv6 address should be followed by a prefix length
- You can use a whole, partially shortened, or shortened address when configuring IPv6 to routers.
![[Pasted image 20240901151430.png]]
`show ipv6 interface brief`
- Same as the IPv4 command, just use `ipv6`.
- Shortened version of the address is displayed.
- Each interface has two IPv6 addresses.
	- **Link-Local Addresses**: automatically configured on an interface when you configure an IPv6 address.
## Modified EUI-64
EUI(Extended Unique Identifier)-64 allows routers to **automatically generate an IPv6 address** by expanding their MAC address to a 64-bit interface ID.
- This interface identifier can then become the 'host portion' of a /64 IPv6 address, which uniquely identifies the interface on a network segment.
- The EUI-64 format allows for the automatic generation of this 64-bit identifier based on the device's MAC address, ensuring a globally unique address.
- 7th bit of the MAC address is called the U/L bit (Universal/Local bit).
	- U/L bit set to 0 = UAA (Universally Administered Address), uniquely assigned to the device by the manufacturer
	- U/L bit set to 1 = LAA (Locally Administered Address), manually assigned by an admin with the `mac-address` command on the interface. Doesn't have to be globally unique.
	- The meaning is reversed in the context of IPv6 addresses/EUI-64.
### EUI-64 Conversion
How to convert the MAC address:
1. Divide the MAC address in half (E.g., `1234 56 | 78 90ab`)
2. Insert FFFE in the middle (E.g., `1234 56|ff fe|78 90ab`)
3. Invert the 7th bit 
	- E.g., 7th bit in `1234 56ff fe78 90ab` is `2`, which is `1` in binary. Invert it into `0`. ![[Pasted image 20240902095314.png]]
Conversion questions:
![[Pasted image 20240902100001.png]]
### EUI-64 Configuration
`ipv6 address 2001:db8::/64 eui-64`
- Configure an IPv6 address, where: 
	- The first 64 bits (network prefix) are configured as given, and 
	- The remaining 64 bits (interface identifier) are automatically generated using the EUI-64 format.
![[Pasted image 20240902100144.png]]
Original MAC address of `g0/0`: `0cf8 2236 8500`.
![[Pasted image 20240902100204.png]]
IPv6 address configured: `2001:db8::ef8:22ff:fe36:8500`.
- Network prefix: `2001:db8::`.
- Interface identifier generated using the EUI-64 conversion: `0ef8 22ff fe36 8500`. (Leading `0` is removed.)
![[Pasted image 20240902101504.png]]
Link local addresses use the same interface ID as the global unicast address that is configured, because both uses EUI-64 to generate the interface ID.
![[Pasted image 20240902110645.png]]
