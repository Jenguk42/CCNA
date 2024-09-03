### Purpose of Classless IPv4 Addressing
- The IANA (Internet Assigned Numbers Authority) assigns IPv4 addresses/networks to companies based on their size.
	- E.g., Large companies may get a class A/B network, and small companies may get a class C network
	- When the Internet was first created, the creators did not predict that the Internet would become as large as today ➡️ resulted in wasted address space 
- Traditional classful IPv4 addresses led to many **wasted IP addresses**
	![[Pasted image 20240816140842.png]]
	- Remember only Class A, B, and C addresses can be assigned to a device as an IP address.
	![[Pasted image 20240816140910.png]]
	- Assigning a class C network to a point-to-point network (A network connecting two points, such as router to router) wastes
		- E.g., Using `203.0.113.0/24` provides 256 addresses 
		- 4 Addresses needed: network address `203.0.113.0`, broadcast address `203.0.113.255`, R1's address `203.0.113.1`, R2's address `203.0.113.2`
		- 252 addresses are wasted!
	- Companies with capacities in between classes will require a bigger class 
		- E.g., Company X needs IP addressing for 5000 end hosts; Class C does not provide enough addresses, so A class B must be used.
		- Results in 60,000 addresses being wasted!
### CIDR (Classless Inter-Domain Routing)
- The IETF (Internet Engineering Task Force) introduced CIDR in 1993 to replace the classful addressing system.
- The requirements of Class A = `/8`, Class B = `/16`, Class C = `/24`, and Class D = `/32` are removed.
- Larger networks can be split into smaller networks called sub-networks (subnets), allowing greater efficiency.
- In a subnet `203.0.113.0/25`:
	- In binary: `1100 1011 . 0000 0000 . 0111 0001 . 0 | 000 0000`
	- Subnetwork bit: 25 bits
		- Host bit is 7 bits (subnetwork bit + host bit = 32)
	- Subnet mask: `1111 1111 . 1111 1111 . 1111 1111 . 1 | 000 0000`
		- = `255.255.255.128` (The last octet `1000 0000` = 128 in decimal)
	- Number of usable address:
		- 2^n - 2 where n = number of host bits
		-  2^7 - 2 = 126 hosts
	- Subnet range:
		- The host bits range from `000 0000` to `111 1111`
		- `... . 0111 0001 . 0 | 000 0000` to `... . 0111 0001 . 0 | 111 1111`
		- = `203.0.113.0` to `203.0.113.127`
	- Network ID: `203.0.113.0` (host bits are `000 0000`)
	- Broadcast: `203.0.113.127` (host bits are `111 1111`)
### VLSM (Variable-Length Subnet Masks)
- Process of creating subnets of different sizes, to make use of network addresses more efficiently
	- DIfferent subnets use different prefix lengths
- Steps are as follows:
	1. Assign the largest subnet at the start of the address.
	2. Assign the second-largest subnet after it.
		-  Network address = broadcast address of the largest subnet + 1
	3. Repeat the process until all subnets have been assigned.
		- Remember to add point-to-point connections.
## Practice Quizzes
### FLSM (Fixed-Length Subnet Masks)
1. You have been given the `172.22.0.0/16` network. You are required to divide the network into 500 separate subnets. What prefix length should you use?
	- If you "borrow" 9 host bits, you have 2 ^ 9 = 512 subnets
	- You should be using 16 + 9 = **25 prefix length**
	
2. What subnet does host `172.25.217.192/21` belong to?
	- The third octet in binary is `1101 1001`
	- 5 host bits are borrowed from the third octet. (16 + 5 = 21)
	- Set all network bits as 0, then the third octet is:
		- `1101 1 | 000` = 216 in decimal
	- So the host belongs to subnet **`172.25.216.0/21`**
	
3. Dividing the `172.30.0.0/16` network into subnets of 1000 hosts each, how many subnets are you able to make?
	- Number of usable addresses: 2^10 - 2 = 1022
	- You need 10 host bits and 6 network bits.
	- You are able to make 2 ^ 6 = **64 subnets**

4. You have been given the `10.0.0.0/8` network. You must create 2000 subnets which will be distributed to  various enterprises.
	1. What prefix length must you use?
		- If you "borrow" 11 host bits, you have 2^11 = 2048 subnets
		- You should be using 8 + 11 = **19 prefix length**
	2. How many host addresses (usable addresses) will be in each subnet?
		- There are 32 - 19 = 13 host bits for each subnet
		- 2^13 - 2 = **8,190 usable addresses** in each subset

5. PC1 has an IP address of `10.217.182.223/11`. Identify the following for PC1's subnet:
	1. Network address:
		- Binary representation of the second octet:  
	2. Broadcast address:
	3. First usable address:
	4. Last usable address:
	5. Number of host (usable) address:
### VLSM (Variable-Length Subnet Masks)
6. A company is assigned the `192.168.1.0/24` network, and must divide it into 5 subnets to provide IP addresses for the enterprise network.  ![[Pasted image 20240816224441.png]]
	- Identify the following for each subnet:
		1. Network address:
		2. Broadcast address:
		3. First usable address:
		4. Last usable address:
		5. Number of host (usable) addresses:
	- Start with Tokyo LAN A (has the most hosts)
		- 2 ^ 7 - 2 = 126 > 110 hosts
			- 7 host bits, 25 network bits; prefix length is 25
		- Network: `192.168. 0000 0001 . 0|000 0000` (=`192.168.1.0/25`)
		- Broadcast: `192.168. 0000 0001 . 0|111 1111` (=`192.168.1.127/25`)
		- First Usable: `192.168. 0000 0001 . 0|000 0001` (=`192.168.1.1/25`)
		- Last Usable: `192.168. 0000 0001 . 0|111 1110` (=`192.168.1.126/25`)
		- 126 hosts usable
	- Then Toronto LAN B (second most hosts)
		- Network address = Broadcast address of the last subnet + 1
			- `192.168. 0000 0001 . 1000 0000` (=`192.168.1.128`)
		- 2 ^ 6 - 2 = 62 > 45 hosts
			- 6 host bits, 26 network bits; prefix length is 26
		- Network: `192.168.1.128/26` (previous broadcast address + 1)
		- Broadcast: `192.168. 0000 0001 . 10|11 1111` (= `192.168.1.191/26`)
		- First Usable: `192.168. 0000 0001 . 10|00 0001` (= `192.168.1.129/26`)
		- Last Usable: `192.168. 0000 0001 . 10|11 1110` (= `192.168.1.190/26`)
		- 62 hosts usable
	- Then Toronto LAN A (third most hosts)
		- Network address = Broadcast address of the last subnet + 1
			- `192.168. 0000 0001 . 1100 0000` (=`192.168.1.192)
		- 2 ^ 5 - 2 = 30 > 29 hosts
			- 5 host bits, 27 network bits; prefix length is 27
		- Network: `192.168.1.192/27` (previous broadcast address + 1)
		- Broadcast: `192.168. 0000 0001 . 110|1 1111` (= `192.168.1.223/27`)
		- First Usable: `192.168. 0000 0001 . 110|0 0001` (= `192.168.1.193/27)
		- Last Usable: `192.168. 0000 0001 . 110|1 1110` (= `192.168.1.222/27)
		- 30 hosts usable
	- Then Tokyo LAN B (least hosts)
		- Network address = Broadcast address of the last subnet + 1
			- `192.168. 0000 0001 . 1110 0000` (=`192.168.1.224)
		- 2 ^ 4 - 2 = 14 > 8 hosts
			- 4 host bits, 28 network bits; prefix length is 28
		- Network: `192.168.1.224/28` (previous broadcast address + 1)
		- Broadcast: `192.168. 0000 0001 . 1110| 1111` (= `192.168.1.239/28`)
		- First Usable: `192.168. 0000 0001 . 1110| 0001` (= `192.168.1.225/28)
		- Last Usable: `192.168. 0000 0001 . 1110| 1110` (= `192.168.1.238/28)
		- 14 hosts usable
	- Then point-to-point connection btw the two routers
		- Network address = Broadcast address of the last subnet + 1
			- `192.168. 0000 0001 . 1111 0000` (=`192.168.1.240)
		- 2 ^ 2 - 2 = 2 hosts
			- 2 host bits, 30 network bits; prefix length is 30
		- Network: `192.168.1.240/30` (previous broadcast address + 1)
		- Broadcast: `192.168. 0000 0001 . 1111 00|11` (= `192.168.1.243/28`)
		- First Usable: `192.168. 0000 0001 . 1111 00|01` (= `192.168.1.241/28)
		- Last Usable: `192.168. 0000 0001 . 1111 00|10` (= `192.168.1.242/28)
		- 2 hosts usable