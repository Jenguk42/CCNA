INCOMPLETE! https://youtu.be/JtQV_0Sjszg?si=mkFt8JU8LFKfk62Q
## DTP
### What is DTP?
Cisco proprietary protocol that allows Cisco switches to dynamically determine their interface status (`access` or `trunk`) without manual configuration
- Enabled by default on all Cisco switch interfaces
- Should be disabled through manual configuration on all switchports (security purposes)
	- Use `switchport mode access` or `switchport mode trunk`
### DTP Configuration
![[Pasted image 20240819202414.png]]
- A switchport in `dynamic desirable` mode will actively try to form a trunk with other Cisco switches. 
	- Will form a **trunk** if connected to another switchport in these modes:
		- `switchport mode trunk`: SW1 negotiates to form a trunk, since SW2 is a trunk  ![[Pasted image 20240819203458.png]]
		- `switchport mode dynamic desirable`: Both switches are actively trying to use DTP to form a trunk ![[Pasted image 20240819203546.png]]
		- `switchport mode dynamic auto`
	- Will form an **access** if connected to an interface in access mode 
		- `switchport mode access`: SW1 agrees to form access connection ![[Pasted image 20240819203719.png]]
- If the operational mode is `static access`, the access port belongs to a single VLAN that doesn't change, unless you configure a different VLAN.
