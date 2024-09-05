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
## DHCP DORA
![[Pasted image 20240905134205.png]]
![[Pasted image 20240905134143.png]]
- `ipconfig /release`
	- Release the DHCP-learned IP address.
- `ipconfig /renew`
	- Get an IP address from the DHCP server.
	- Involves four messages.
	1. **DHCP Discover** message: Client searches for DHCP server in the network, telling them it needs an IP address. ![[Pasted image 20240905132636.png]]
		- Broadcast message
			- Destination MAC address is `ffff:ffff:ffff`
			- Destination IP address is `255.255.255.255`
		- Source IP: `0.0.0.0`, the PC doesn't have an IP address yet.
		- Source port 68 (client), Dest port 67 (server)
		- Bootp flags: `0x0000` unicast, the PC is asking the server to send its messages using unicast.
		- Option 50: Requested for the same IP address as before.
	2. **DHCP Offer** message: DHCP server offers the client some information such as IP address, default gateway, DNS server, etc. 
		- DHCP Offer message can be either broadcast or unicast.
		- Options include the different information.
	3. **DHCP Request** message: Client tells server that it wants to use the IP address it was offered.
		- The client has to tell which server it is accepting the offer from when requesting to use an IP address.
			- This is because there may be multiple DHCP servers on the local network, and all of them will reply to the client's Discover message with an Offer. 
		- Typically, the client will receive the first offer it receives.
	4. **DHCP Ack** message: Server confirms that the client may use the requested IP address. 
		- Once this is received, the client can configure the IP address on its interface.
		- DHCP Ack message can be either broadcast or unicast.
## DHCP Relay
Large enterprises often choose to use a **centralised DHCP server**, which will assign IP addresses to DHCP clients in all subnets throughout the enterprise network.
- A router should be configured as a **DHCP relay agent** to forward the clients' broadcast DHCP messages to the remote DHCP server as unicast messages.
- This is because broadcast messages don't leave the local subnet, and there should be a way to forward the DHCP messages.

In the network topology below, SRV1 is a DHCP server and R1 is a DHCP relay agent.
![[Pasted image 20240905135126.png]]
1. PC1 broadcasts a DHCP discover message to get an IP address.
2. R1 relays the message to SRV1. (Src and Dst IP addresses change to unicast addresses.)
3. SRV1 replies with the DHCP offer, and R1 relays it to PC1.
4. PC1 replies with the DHCP request, and R1 relays it to SRV1.
5. SRV1 replies with the DHCP acknowledgement, and R1 configures the IP address.
## DHCP Configuration on Cisco Routers
![[Pasted image 20240905140220.png]]
- `show ip dhcp binding`
	- Shows all DHCP clients that are currently assigned with IP addresses.
## DHCP Server Configuration
![[Pasted image 20240905140309.png]]
- `ip dhcp excluded-address ADDRESS_START ADDRESS_END`
	- Specify a reserved range, assigned to servers, network devices, etc.
- `ip dhcp pool POOL_NAME`
	- Create a DHCP pool
	- DHCP pool: a subnet of addresses that can be assigned to DHCP clients, as well as other info such as DNS server and default gateway.
	- **Separate DHCP pool** should be configured for each network the router is acting as a DHCP server for.
- `network NETWORK_ADDRESS [NETWORK_MASK | PREFIX_LENGTH]`
	- Actual configuration of the range of addresses to be assigned to clients. (Addresses reserved above will not be used.)
	- Both network masks (`255.255.255.0`) or prefix length (`/24`) work.  
- `dns-server DNS_SERVER`
	- Configuration of the DNS server the clients should use.
- `domain-name DOMAIN_NAME` = specify the domain name of the network.
- `default-router DEFAULT_GATEWAY` = specify the default gateway.
- `lease DAYS HOURS MINUTES` or `lease infinite` = specify the lease time.
## DHCP Relay Agent Configuration
![[Pasted image 20240905140552.png]]
- `ip helper-address DHCP_SERVER`
	- Set the IP address of the DHCP server as the 'helper' address.
	- Should be configured on the interface connected to the subnet of the CLIENT DEVICES.
	- Make sure that a route to the DHCP server is present. 
		- Static routing or OSPF can be used.
## DHCP Client Configuration
![[Pasted image 20240905141047.png]]
- `ip address dhcp`
	- Broadcast a DHCP Discover message and get an IP address from the server.
	- Run on interface configuration mode (on the interface connected to the server)
	- Usually, network devices would be configured with a fixed IP address.
## Command Summary
![[Pasted image 20240905141111.png]]