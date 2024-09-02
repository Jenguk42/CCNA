Layer 4 protocols, be able to compare the two!
## Basics of Layer 4
Layer 4 provides transparent transfer of data between end hosts.
![[Pasted image 20240830140214 1.png]]
- It encapsulates the data with a Layer 4 header, then uses the services of the lower layers (Layers 3, 2, and 1) to deliver the data unchanged to the destination host.
    - The hosts are not aware of the details of the underlying network - the transfer of data is 'transparent' to them.
- Provides (TCP) or doesn't provide (UDP) various services to applications, such as:
	- **Reliable data transfer**: Making sure the destination host actually received every bit of data.
	- **Error recovery**: Ensuring the data is sent again if an error happened in transmission.
	- **Data sequencing**: Ensuring the end host can sequence the data in correct order even if they arrive out of order.
- Provides Layer 4 addressing (**port** numbers)
	- Used for both TCP and UDP.
	- These port numbers can:
		- Identify the Application Layer protocol
	        - Provide [[#Session Multiplexing]].
- Port numbers that the Application Layer protocols use are registered with the IANA (Internet Assigned Numbers Authority).
	- **Well-known** port numbers: **0 - 1023**
		- Used for major protocols like HTTP, FTP, etc, and are strictly regulated.
	- **Registered** port numbers: **1024 - 49151**
		- Registration is required, but it is not strict.
	- **Ephemeral/private/dynamic** port numbers: **49152 - 65535**
		- Hosts use this range to select the random source port.
### Session Multiplexing
Session: An exchange of data between 2+ communicating device
- A PC should be able to handle multiple communication sessions at once. (E.g., Multiple internet tabs open)

Example Topology: PC1 is communicating with SRV1. ![[Pasted image 20240830150730.png]]
- PC1 uses TCP at Layer 4, with a source port of `50000` and a destination port of `80`.
	- **Destination port** identifies the Application Layer Protocol. (E.g., TCP port 80 is used for the protocol HTTP, which is used to access websites.)
	- **Source port** is randomly selected by PC1, and helps to identify the session (in combination with the destination port).
- In SRV1's reply, the source and destination port numbers are reversed.
    - The source & destination port numbers tell PC1 that the reply is **part of the same communication session** as the message it sent earlier.
 
In a separate connection to SRV1, PC1 uses a different source port.
![[Pasted image 20240830151102.png]]
- SRV1 will use that source port as the destination port for its response, so PC1 knows it is part of that session.

Same thing happens when PC1 wants to access SRV2 at the same time.
![[Pasted image 20240830151201.png]]
- This time FTP protocol is used, which is shown by a TCP destination port number of 21.
	- Source port was randomly selected as 60000.
	- SRV2's reply will reverse the port numbers, using 21 as source and 60000 as destination.
## TCP (Transmission Control Protocol)
Overview of the services provided:
- **Connection-oriented**: A connection has to be established between the two hosts, before actually sending data. 
	- Achieved by [[#Establishing Connection Three-Way Handshake]] and [[#Terminating Connection Four-Way Handshake]]
	- Once the connection is established, the data exchange begins.
- **Reliable communication**: The destination host must acknowledge that it received each TCP segment.
	- Achieved by [[#Forward Acknowledgement]] and [[#TCP Retransmission]]
- **Sequencing**: Sequence numbers of the TCP header allow destination hosts to put segments in the correct order even if they arrive out of order.
	- Achieved by [[#Forward Acknowledgement]]
- **Flow Control**: The destination host can tell the source host to increase/decrease the rate that data is sent.
	- Achieved by [[#Window Size Control]]
### TCP Header
![[Pasted image 20240830152402.png]]
- Source and destination port: 16 bits, 2 bytes in length
	- There are a total of 65536 (2^16) available port numbers.
- Sequence number & ACK number: provides sequencing and reliable communication
- The following flag bits are used to establish and terminate connections:
	- `ACK` (acknowledgement), `SYN` (Synchronisation), `FIN`
- Window Size: Used for flow control, adjusting the rate at which data is sent.
### Establishing Connection: Three-Way Handshake
A method TCP uses to **establish** connections. Involves three messages being sent between the two hosts.
![[Pasted image 20240830153326.png]]
Example Topology: PC1 wants to connect to SRV1.
1. PC1 sends a TCP segment to SRV1 with the `SYN` flag set to 1.
2. SVR1 replies by sending a TCP segment to PC1 with the `SYN` and `ACK` flags set to 1.
3. PC1 sends a TCP segment to SRV1 with the `ACK` flag set to 1.
Now the three-way handshake is complete and the connection is established.
### Terminating Connection: Four-Way Handshake
A method TCP uses to **terminate** connections. Involves four messages being sent between the two hosts.
![[Pasted image 20240830153622.png]]
Example Topology: PC1 wants to end the connection with SRV1.
1. PC1 sends a TCP segment to SRV1 with the `FIN` flag set to 1.
2. SRV1 responds with an `ACK`.
3. SRV1 then sends its own `FIN`.
4. PC1 sends an ACK in response to SRV1's FIN.
Now the four-way handshake is complete and the connection is terminated.
### Forward Acknowledgement
TCP uses **forward acknowledgement** to indicate the sequence number of the next segment the host expects to receive.
![[Pasted image 20240830154643.png]]
- Hosts set a random initial sequence number.
- Acknowledgement field = previous Sequence field + 1
	- E.g., PC2 tells PC1 the sequence number of **the next segment it expects to receive**, which is 11 (Not 10).
- Sequence field = previous Acknowledgement field
### TCP Retransmission
If the source host does not receive an acknowledge for a segment, the segment is sent again.
![[Pasted image 20240830155137.png]]
- PC1 waits after sending Seq 21, but there is no Ack coming from SRV1. After waiting a certain amount of time, it resends the segment.
### Window Size Control
![[Pasted image 20240830155646.png]]
Acknowledging every single segment is inefficient, no matter the size.
- The TCP header's **Window Size** field allows more data to be sent before an acknowledgement is required.
	- E.g., Three segments can be sent with sequence numbers 20, 21, and 22, then Ack can be sent with a sequence number of 23.
- A 'sliding window' can be used to dynamically adjust how large the window size is.
	- The window size is increased as much as possible until a segment is dropped, then it backs down to a more reasonable level, and slowly increases again.
## UDP (User Datagram Protocol)
![[Pasted image 20240830160055.png]]
- Four fields:
	- Source and destination port
	- Length field indicating the length of the segment
	- Checksum for the receiving host to check for errors
- **Not** connection oriented.
	- The sending host does not establish a connection with the destination host before sending data. The data is simply sent.
- Reliable communication **not** provided.
	- When UDP is used, acknowledgements are not sent for received segments.
	- If a segment is lost, UDP has no mechanism to re-transmit it.
	- Segments are sent 'best-effort' - there is no guarantee of delivery.
- Sequencing **not** provided.
	- There is no sequence number field in the UDP header.
	- If segments arrive out of order, UDP has no mechanism to put them back in order.
- Flow control **not** provided.
	- UDP has no mechanism like TCP's window size to control the flow of data.
## TCP vs. UDP
Both TCP and UDP provide Layer 4 addressing in the form of port numbers, which identify Application Layer protocols and allow for session multiplexing.
![[Pasted image 20240830160542.png]]
- **TCP** provides more features than UDP, but at the cost of additional overhead.
	- Acks and retransmissions can slow down the transfer of data.
	- Preferred for applications that require **reliable communications** (E.g., downloading a file).
- **UDP** is preferred for applications that are **delay-sensitive.**
	- E.g., Real-time voice and video
	- Some applications that use UDP can provide reliability within the application itself. (E.g., TFTP, Trivial File Transfer Protocol)
- There are some applications that use both TCP & UDP, depending on the situation.
	- E.g., DNS, Domain Name System
## Important Well-known Port Numbers
Remember that DNS uses both TCP and UDP, depending on the situation.
![[Pasted image 20240830160825.png]]