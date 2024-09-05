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
1. Managed devices can **notify the NMS of events**.
	- E.g. SW1 to SRV1: "My Interfaces is going up!"
2. The **NMS can ask the managed devices for information** about their current status. (The devices will reply.)
	- E.g. SRV1 to R1: "What's your current CPU usage?"
3. The **NMS can tell the managed devices to change aspects of their configuration**. (The devices will comply and report.)
	- E.g., SRV1 to R1: "Change your IP address of G0/1 to `203.0.113.5/30`."
## How does It Work?
![[Pasted image 20240905154006.png]]
- **NMS**
	- Green section: SNMP software on the NMS.
		- SNMP is just a part of the NMS.
	- **SNMP Manager**: The software on the NMS that interacts with the managed devices
		- Receives notifications, sends requests for info, sends configuration changes, etc.
		- **Listens to messages on port UDP 162.**
	- **SNMP Application**: Provides an interface for the network admin to interact with
		- Displays alerts, statistics, charts, etc.
- **Managed Devices**
	- Blue section: SNMP entity on the managed devices.
	- **SNMP Agent**: The software on the managed devices that interacts with the SNMP Manager on the NMS.
		- Sends notifications to/ receives messages from the NMS.
		- **Listens to messages on port UDP 161.**
	- **MIB (Management Information Base)**: Structure that contains the variables that are managed by SNMP.
		- Each variable is identified with an Object ID (OID).
		- Examples of variables: interface status, traffic throughput, CPU usage, device temperature, etc. 
- SNMP messages exchanged between the SNMP Manager and Agent make SNMP work.
## SNMP OIDs
SNMP OIDs are organised in an hierarchical structure.
![[Pasted image 20240905154923.png]]![[Pasted image 20240905154944.png]]
For more information on SNMP OIDs, check the [OID Repository](https://oid-info.com).
## SNMP Versions
Many versions of SNMP have been proposed/developed, but only the following three major versions are used widely:
- **SNMPv1**
	- Original version of SNMP.
- **SNMPv2c**
	- Most widely used.
	- Allows the NMS to retrieve large amounts of information in a single request, so it's more efficient.
	- 'c' refers to the 'community strings' used as passwords in SNMPv1, removed from SNMPv2, then added back to SNMPv2c.
- **SNMPv3**
	- Best version so far. Use this version whenever possible!
	- A much more secure version of SNMP that supports strong encryption and authentication.
## SNMP Message Types
![[Pasted image 20240905155923.png]]
- Read Class
	- `Get`: A request sent from the manager to the agent to retrieve the value of a variable (OID), or multiple variables.
		- The agent will send a `Response` message with the current value of each variable. 
		- Example: ![[Pasted image 20240905160102.png]]
	- `GetNext`: A request sent from the manager to the agent to discover the available variables in the MIB.
		- "Tell me the next OID, so I can discover what other OIDs are available."
	- `GetBulk`: A more efficient version of the `GetNext` message, introduced in SNMPv2.
- Write Class
	- `Set`: A request from the manager to the agent to change the value of one or more variables.
		- The agent will send a `Response` message with the new values.
		- Example: ![[Pasted image 20240905160325.png]]
- Notification Class
	- `Trap`: A notification sent from the agent to the manager. 
		- Unreliable because the manager does NOT send a Response message to acknowledge that it received the Trap.  
		- SNMP uses UDP, so there are no TCP retransmissions either.
		- Example: This trap will appear as an alert on the SNMP application of SRV1. ![[Pasted image 20240905160630.png]]
	- `Inform`: A notification message that is acknowledged with a Response message.
		- Sent with UDP, but there is reliability built into the message itself.
		- Originally used for communications between managers, but later updates allow agents to send Inform messages to managers, too.
## SNMPv2c Configuration
16:51 https://www.youtube.com/watch?v=HXu0Ifj0oWU&list=PLxbwE86jKRgMpuZuLBivzlM8s2Dk5lXBQ&index=78