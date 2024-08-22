Allows devices to configure speed and duplex settings without manual configuration

* Interfaces that can run at different speeds (10/100 or 10/100/1000) have default settings of **speed auto** and **duplex auto**.
* Interfaces 'advertise' their capabilities to the neighbouring device, and they negotiate the best **speed** and **duplex** settings they are both capable of.

![[Pasted image 20240813195751.png]]
* Ethernet interface can run at 10 Mbps,
* Fast Ethernet interface can run at 10/100 Mbps,
* Gigabit Ethernet interface can run at 10/100/1000 Gbps

### If autonegotiation is disabled?
* SPEED: Switch will try to sense the speed that the device is operating at
	* If it fails to sense the speed, it will use the slowest supported speed
	* E.g., 10 Mbps on a 10/100/1000 interface
* DUPLEX: Switch will choose full/half duplex based on the speed
	* If speed is 10/100, half duplex
	* If speed is 1000 or greater, full duplex
* Autonegotiation is disabled below:
	![[Pasted image 20240813200050.png]]
	* Because the switch senses that the blue PC has a speed of 100 Mbps, it will set the duplex as half
	* Duplex mismatch due to collisions -> poor network performance
	* Use autonegotiation for all devices in the network!
