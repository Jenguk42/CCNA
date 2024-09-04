## LAN
* A single **broadcast domain**, including all devices in that broadcast domain.
	* A **broadcast domain** is the group of devices which will receive a broadcast frame (destination MAC `FFFF.FFFF.FFFF.FFFF`) sent by any one of the members.
	* Router to router connection is still technically a broadcast domain
* Network contained within a relatively small area
	* *E.g., Office floor, home network*
* Routers connect separate LANs
* Switches connect and expand networks
	* Adding more switches does not separate networks
	* Switches connected to different router interfaces?
		* Two separate LANs (LANs 3 and 4)
		![[Pasted image 20240812155547.png]]
- Security & performance concerns - what if I have a LAN of a company, and want separate networks for the Engineering, HR, and Sales team?
	- Refer to [[VLANs]]
## Ethernet Frame
![[Pasted image 20240812155632.png]]
* 5 fields in the header
* **Preamble, SFD**: Used for synchronisation and to allow the receiving device to be prepared to receive the rest of the data in the frame
	* ==**Preamble**==: 7 bytes (56 bits), alternating 1's and 0's that allow devices to synchronise their receiver clocks: **10101010 * 4**
	* ==**SFD**== (Start Frame Delimiter): 1 byte (8 bits), alternating 1's and 0's that ends with 1; marks the end of the preamble and beginning of the rest of the frame: **101011**
* ==**Destination**==: Layer 2 address to which the frame is being sent
	* Indicates the destination [[MAC address]]
* ==**Source**==: Layer 2 address of the device that sent the frame
	* Indicates the source [[MAC address]]
* ==**Type**==: Layer 3 protocol used in the encapsulated packet (IPv4 or IPv6)
	* Can also be ==**length**==: Length of the encapsulated data
* ==**FCS**==  (Frame Check Sequence): Receiving device uses this to detect any errors that might have occurred in transmission
* **802.1Q tag** is inserted between the source MAC address and the Type field
	* Used for VLAN tagging, refer to [[VLANs]]

* Lengths of the ethernet frame
	* **Preamble + SFD** usually not considered part of the Ethernet header
	* Min size for an Ethernet frame is 64 bytes
		* Header + Payload (Packet) + Trailer = **64 bytes**
		* Ethernet header + trailer: **18 bytes** (6+6+2+4)
		* Min payload (packet) size is **46 bytes** (64 - 18)
	* If payload < 46 bytes, padding bytes are added
		* *E.g., 34-byte packet + 12-byte padding = 46 minimum bytes*

Broadcast is limited at the router