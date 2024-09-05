Dynamic Host Configuration Protocol
## Purpose of DHCP
Allows hosts to automatically/dynamically learn various aspects of their network configuration, without manual/static configuration.
- Aspects include IP address, subnet mask, default gateway, DNS server, etc.
- Essential part of modern, large networks!
	- Once connected to Wi-Fi, we gain automatic access to the internet - no need to ask the network admin for the IP address, subnet mask, default gateway, etc.
- Typically used for 'client device' (E.g., PCs, phones, etc.)
- Devices such as routers, servers, etc. are manually configured.
	- These devices need a fixed setting to perform their function.
- In small networks (such as home networks), the router typically acts as the DHCP server for hosts in the LAN.
- In larger networks, the DHCP server is usually a Windows/Linux server.
## Basic Functions of DHCP
![[Pasted image 20240905130740.png]]
- DHCP Enabled: Shows that the PC is auto-configured by DHCP.
- IPv4 Address is `Preferred`
	- The PC was previously assigned this IP address by the DHCP server, so it asked to receive the same address again.
	- If the address was not available, the preferred address may not have been granted.
- **Lease** Obtained & Expires
	- Lease times should be limited to preserve the available IP addresses.
		- E.g., You don't want the customers' devices to be permanently assigned to an IP address when connected to a Café Wi-Fi
	- DHCP server 'lease' IP address to clients.
	- These leases are usually not permanent, and the client must give up the address at the end of the lease.
	- A client can also release the address before the lease is up, if it no longer needs it. 
- Default gateway, DHCP server, and DNS servers are all configured as home routers - this is common in home networks.
- UDP port numbers:
	- DHCP servers use UDP 67
	- DHCP clients use UDP 68
## PC Configuration

- `ipconfig /release`
	- Release the DHCP-learned IP address.
	- DHCP release message shown in Wireshark: ![[Pasted image 20240905131921.png]]