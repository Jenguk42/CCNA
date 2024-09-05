Domain Name System, allows easier access to websites
- Domain Name: Defines an area of administrative control in the internet.
	- E.g., Gmail at 'mail.google.com', NTP servers at 'time.google.com', etc. fall under Google's administration and control.
## Purpose of DNS
Used to *resolve* (convert) human-readable names (google.com) to IP address.
- Machines such as PCs don't use names, they use addresses.
- Names are much easier for humans to use and remember than IP addresses!
	- E.g., When you type "youtube.com" into a web browser, your device will ask a DNS server for the IP address of youtube.com.
- DNS server(s) your device uses can be manually configured or learned via DHCP (Dynamic Host Configuration Protocol).
## How it Works
![[Pasted image 20240905093231.png]]
- From PC1 (Windows PC):
	- `ipconfig /all` displays various info on PC1. ![[Pasted image 20240905093455.png]]
	- `nslookup NAME` 
		![[Pasted image 20240905093920.png]]
		- Stands for "Name Server Lookup"
		- PC1 asks the DNS server for the IP address of the specified name.
			- PC1: What's the IP address of youtube.com?
			- dns.google.com: It's `172.217.25.110`.
			- R1: **No DNS configuration required**, just forwards packets.
- Wireshark screenshot:
	![[Pasted image 20240905094233.png]]
	- DNS `A` record = Used to map names to IPv4 addresses.
	- DNS `AAAA` (quadruple A) record = Used to map names to IPv6 addresses. 
	- Standard DNS queries/responses typically use **UDP**.
		- **TCP** is used for DNS messages greater than 512 bytes.
		- In either case, port 53 is used.
## DNS Cache
Devices will save the DNS server's response to a local DNS cache, so they don't have to query the server every time they want to access a particular destination.
- `ipconfig /displaydns` displays the DNS cache.
	![[Pasted image 20240905094832.png]]
	- `CNAME` = canonical name, another kind of DNS record that maps the DNS to another name.
- `ipconfig /flushdns`
	- Clears the DNS cache.
	- E.g., Another DNS query should be sent to learn the IP address of youtube.com.
## Host File
Alternative to DNS; Most devices have a 'host' file, which is a list of hosts and IP addresses. (DNS is a much better solution!)
![[Pasted image 20240905095411.png]]
- In windows, the path is `C:\Windows\System32\drivers\etc\hosts`
- There is nothing by default, but entries can be added by typing the IP address, a space, and the host name.
- E.g., `192.168.0.1 R1`
	- Because PC1 has an entry of R1 in its hosts file, it will be able to ping R1.
## Configuring DNS in Cisco IOS
For hosts in a network to use DNS, you don't need to configure DNS on the routers. They will simply forward the DNS messages like any other packets.
- However, a Cisco router can be configured as a DNS server, although it's rare.
- If an internal DNS server is used, it's usually a Windows or Linux server.
	- Internal DNS server: DNS server in the local network.
	- Public DNS server: DNS server accessible through the internet, like Google's.
- A Cisco router can also be configured as a DNS client. 
- All config commands are run in global config mode!
### DNS Show Command
![[Pasted image 20240905100953.png]]
- `show hosts`
	- View both the configured hosts, as well as the hosts learned and cached via DNS.
	- `temp`: Temporary - it is learned by DNS, so it will expire. 
	- `perm`: Permanent - it is manually configured, so it will not expire.
### DNS Server Configuration
![[Pasted image 20240905093231.png]]
![[Pasted image 20240905100320.png]]
- `ip dns server`
	- Run on global config mode
	- Configures the router to act as a DNS server.
	- If a client sends a DNS query to R1, R1 will respond if it has the requested record.
- `ip host HOST_NAME IP_ADDRESS`
	- Build a host table on the router.
- `ip name-server DNS_SERVER`
	- Configure an external DNS server that the router can query if it doesn't have the requested record in its host table.
- `ip domain lookup`
	- Allows the router to perform DNS queries on the external server.
	- Enabled by default!
### DNS Client Configuration
![[Pasted image 20240905101811.png]]
`ip dns server` is not run, so R1 is not configured as a server. Now it will use the specified DNS server to perform queries (E.g., ping youtube.com).
- `ip domain name NAME`
	- Configures the default domain name.
	- This will be automatically appended to any hostnames without a specified domain name.
	- E.g., `ping pc1` will become `ping pc1.jeremysitlab.com`
### Command Overview
![[Pasted image 20240905102248.png]]
## Demonstration
![[Pasted image 20240905100549.png]]
- PC1 pings PC2 by consulting R1.

![[Pasted image 20240905100817.png]]
- PC1 tries to ping youtube.com by consulting R1. 
	- However, R1 doesn't have an entry for it. 
	- So it consults the external DNS server (google) for the IP address. 
	- Google's server replies to R1, R1 replies to PC1, then PC1 pings YouTube.
