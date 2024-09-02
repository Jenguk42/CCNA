Layer 4 protocols, be able to compare the two.
## Basics of Layer 4
- Layer 4 provides transparent transfer of data between end hosts.
	![[Pasted image 20240830140214.png]]
	- It encapsulates the data with a Layer 4 header, then uses the services of the lower layers (Layers 3, 2, and 1) to deliver the data unchanged to the destination host.
	- The hosts are not aware of the details of the underlying network - the transfer of data is 'transparent' to them.
- Provides (TCP) or doesn't provide (UDP) various services to applications, such as:
	- Reliable data transfer: Making sure the destination host actually received every bit of data.
	- Error recovery: Ensuring the data is sent again if an error happened in transmission.
	- Data sequencing: Ensuring the end host can sequence the data in correct order even if they arrive out of order.
	- Flow control: Making sure that the source host doesn't send traffic faster than the destination host can handle.
- Provides Layer 4 addressing (**port** numbers)
	- These port numbers can:
		- Identify the Application Layer protocol
		- Provide session multiplexing 
### Session Multiplexing
- Session: An exchange of data between 2+ communicating devices
## TCP (Transmission Control Protocol)

## UDP (User Datagram Protocol)

## TCP vs. UDP