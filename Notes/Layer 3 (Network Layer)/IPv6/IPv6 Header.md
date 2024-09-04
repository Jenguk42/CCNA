![[Pasted image 20240902152511.png]]
Much simpler, because it is a "fixed" header format - IPv6 header has a fixed size of 40 bytes.
- This is why there is a 'Payload length' field that indicates the length of the encapsulated Layer 4 segment, but no 'header length' field like IPv4.
- Easier process - improved performance.
### Version
- Length: 4 bits
- Indicates the version of IP that is used.
- Fixed value of 6 (`0b0110`) to indicate IPv6.
### Traffic Class
- Length: 8 bits
- Used for QoS (Quality of Service), to indicate high-priority traffic.
- E.g., IP phone traffic, live video calls, etc. will have a Traffic Class value which gives them priority over other traffic.
### Flow Label
- Length: 20 bits
- Used to identify specific traffic 'flows' (communications between a specific source and destination).
### Payload Length
- Length: 16 bits
- Indicates the length of the payload (the encapsulated Layer 4 segment) in bytes.
- The length of the IPv6 header is not included, because it's always 40 bytes.
- E.g., 1024 in this field means that the encapsulated segment is 1024 bytes in length.
### Next Header
- Length: 8 bits
- Indicates the type of the 'next header' (header of the encapsulated segment).
- Same function as the IPv4's 'Protocol' field.
- E.g., TCP, UDP
### Hop Limit
- Length: 8 bits
- The value in this field is decremented by 1 for each router that forwards it. If this reaches 0, the packet is discarded.
- Same function as the IPv4's 'TTL' field.
### Source & Destination Address
- Length: 128 bits each
- These fields contain the IPv6 addresses of the packet's source and the packet's intended destination.