Simple Network Management Protocol (SNMP) is a framework and protocol originally released in 1988. 
- SNMPv1 consists of:
	- RFC 1065 - Structure and identification of management information for TCP/IP-based internets
	- RFC 1066 - Management information base for network management of TCP/IP-based internets
	- RFC 1067 - A simple network management protocol
## Purpose and Basics
Used to monitor the status of devices, make configuration changes, etc.
Two main types of devices:
1.  Managed Devices
	- Devices being managed using SNMP
	- E.g., network devices like routers and switches
2. Network Management Station (NMS)
	- Device/Devices managing the managed devices
	- The SNMP 'server'
## SNMP Operations
![[Pasted image 20240905150249.png]]
Three main operations used in SNMP:
1. Managed devices can notify the NMS of events.
	- E.g. SW1 to SRV1: "My Interfaces is going up!"
2. The NMS can ask the managed devices for information about their current status. (The devices will reply.)
	- E.g. SRV1 to R1: "What's your current CPU usage?"
3. The NMS can tell the managed devices to change aspects of their configuration. (The devices will comply and report.)
	- E.g., SRV1 to R1: "Change your IP address of G0/1 to `203.0.113.5/30`."
## How does It Work?
6:25 https://www.youtube.com/watch?v=HXu0Ifj0oWU&list=PLxbwE86jKRgMpuZuLBivzlM8s2Dk5lXBQ&index=77