- `ipconfig`
	![[Pasted image 20240830095227.png]]
	- Shows the IP configuration settings
- `tracert IP_ADDRESS`
	- The command is `traceroute IP_ADDRESS` in Cisco IOS
	- Sends a ping, and every Layer 3 'hop' along the route to the destination sends a message back to the source.
	- Example: PC1 is sending a ping to `10.0.2.1` through R1, SPR1, SPR2, R2, and SRV1 (Refer to [Day 24 Lab](https://youtu.be/KuKC0G3LZc8?si=NadxYOJSACKHrRsS)) ![[Pasted image 20240823121206.png]]![[Pasted image 20240823121015.png]]
- `ping -t DESTINATION_ADDRESS`
	- Tells the PC to ping continuously until it is told to stop.
