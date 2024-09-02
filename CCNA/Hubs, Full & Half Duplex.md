A hub floods any traffic it receives
Operates at layer 1, repeating whatever signals they receive
![[Pasted image 20240813160224.png]]
* Just like what the switch does when receiving an unknown unicast frame
### Collision Domain
This results in a problem when there is a two way communication
![[Pasted image 20240813160600.png]]
* PC1 sending a frame to PC2, PC3 sending a frame to PC1
* The hub tries to flood both frames at the same time, which will result in a **collision** on the interface
* PC2 will not receive either frame
* All devices within the reach of a hub is within a **collision domain**
- Switches create separate collision domains
	![[Pasted image 20240813194343.png]]
	- Uses layer 2 addressing (MAC addresses) to send frames to the specific host
	* Doesn't send two frames to the same host at once
	* Can operate at full-duplex
### Full & Half Duplex
- **Half Duplex:** The device cannot send & receive data at the same time.
	* If it is receiving a frame, it must wait before sending a frame.
	* Devices attached to hubs must operate in half duplex
	* Not widely used in modern network
- **Full Duplex**: The device can send & receive data at the same time, it does not have to wait.
	- Devices attached to a switch operate a full duplex
### CSMA/CD
Carrier Sense Multiple Access with Collision Detection
-  Devices 'listen' to the collision domain until they detect that other devices are not sending.
- If a collision does occur, the device sends a jamming signal to inform the other devices that a collision happened.
- Each device will wait a random period of time before sending frames again.
- Repeat the process.