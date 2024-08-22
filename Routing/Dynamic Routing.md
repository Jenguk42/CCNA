Configure a dynamic routing protocol on the router, and let the router take care of finding the bets routs to destination networks.
- E.g., Routers will automatically inform each other how t o reach a new destination network.
## Example Topology
![[Pasted image 20240822145634.png]]
Routes that are automatically added without configuration:
- Network route: A route to a network/subnet (mask length < 32)
	- `10.0.12.0/30`, `10.0.13.0/30`
- Host route: A route to a specific host (/32 mask)
	- Specific addresses configured to R1's `G0/0` and `G0/1` interfaces
	- `10.0.12.1/32`, `10.0.13.1/32`
## Dynamic Routing Protocol
When a dynamic routing protocol is enabled, the routers will 'advertise' the reachable networks to its neighbour. The routing tables will be updated accordingly.
![[Pasted image 20240822150240.png]]
If there is a connection error, the routing table will be adjusted and unreachable route will be removed.
- Prevents the router from continuously sending traffic to a dead-end.
## Dynamic Routing Protocol Metrics
## Administrative Distance