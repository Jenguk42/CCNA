How is traffic forwarded between different LANs?
* Layer 3 (Network) layer of the OSI model
### IP Addresses
* Series of 32 bits split up into 4 octets (8 + 8 + 8 + 8)
* Each octet can range from 0 to 255 (00000000 to 11111111)
* Each bit is binary, but we write them using dotted decimal form
	* E.g., 192.169.1.0 = 11000000.10101001.00000001.00000000
### IPv4 Addresses Classes
![[Pasted image 20240813110238.png]]
![[Pasted image 20240813111148.png]]
* The **prefix length** tells what part of the IP address represents the network and the end hosts
	* Prefix length is /24, then:
		* The first 24 bits (the first 3 octets) are the network portion
		* The last 8 bits (the last octet) is the host portion
	* Prefix length is different for each class
		* Class A: ==12==.128.251.23/8
			* Fewer possible network addresses, but can have many hosts on each network
		* Class B: ==154.78==.111.32/16
		* Class C: ==192.168.1==.254/24
			*  Many possible network addresses, but fewer hosts on each network
* Cisco devices use a dotted decimal netmask instead of the slash notation
	* Class A: /8, 255.0.0.0 (=11111111 00000000 00000000 00000000)
	* Class B: /16, 255.255.0.0 (= 11111111 11111111 00000000 00000000)
	* Class C: /24, 255.255.255.0 (= 11111111 11111111 11111111  00000000)
* Class A's usable range is considered to be 1 to 126 
	* Range 127.0.0.0 to 127.255.255.255 is reserved for **loopback addresses**
		* Used to test the network stack (OSI, TCP/IP model) on the local device
		* When ping is sent, PC sends and receives the ping  to and from itself
* Class D addresses  (1110xxxx, 224-239) are reserved for 'multicast' addresses
* Class E addresses (1111xxxx, 240-255) are reserved for experimental use 
### Network and Broadcast Addresses
![[Pasted image 20240813113410.png]]

* Host portion of the address is all 0's = Network Address
	* E.g., 192.168.1.0/24
	* Used to identify the address of the network itself
* Host portion of the address is all 1's = Broadcast Address
	* E.g., 192.168.1.255/24
	* Used to send network traffic to all hosts on the network
	* This means if Dst. IP is 192.168.1.255, Dst. MAC would be FFFF.FFFF.FFFF
* These two addresses cannot be assigned to a host, so **maximum hosts per network = 2^n - 2 (n = number of host bits)**
### IPv4 Addresses within a Network
![[Pasted image 20240813092937.png]]
* Networks are separated by the router
* First three groups (192.168.1, 192.168.2) represent the network itself
* The last 0 changes to represent the end hosts (PCs)
	* PC1: 192.168.1.**1**, PC2: 192.168.1.**2**, PC3: 192.168.2.**1**, PC4: 192.168.2.**2**
* Router needs an IP address for each network it is connected to
	* G0/0 interface: 192.168.1.254
	* G0/1 interface: 192.168.2.254