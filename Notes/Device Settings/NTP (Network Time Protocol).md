INCOMPLETE!! https://youtu.be/qGJaJx7OfUo?si=a9xUSG0u_f8whW3R 23:57

All devices (routers, switches, PC, etc.) have an internal clock, and it's important that **all devices are synchronised** with each other.
- Can be viewed by the `show clock` command.
	- The default time zone is UTC (Coordinated Universal Time).
- `show clock detail` command shows the time source.
	![[Pasted image 20240904144633.png]]
	- Hardware calendar: Built-in internal clock of the device, default time source 
		- Will drift over time, so it's not the ideal time source.
	- `*`: Time is not considered authoritative, and the device is not confident this time is accurate.
The most important reason to have accurate time on a device is to **have accurate logs for troubleshooting**, which is critical for troubleshooting.
- `Syslog`: The protocol used to keep device logs.
- `show logging` shows a list of things happened in different devices. Timestamps should match to correlate the logs of the devices.
## Manual Time Configuration
![[Pasted image 20240904153218.png]]
### Hardware & Software Clock Configuration
Although the hardware calendar (built-in clock) is the default time-source, the hardware clock and software clock are separate and can be configured separately.
- Run from the privileged-exec mode, not global config! This is because the clock config is not a part of the device's running configuration.
- `clock set CURRENT_TIME DAY MONTH YEAR`
	- E.g., `clock set 14:58:00 4 Sep 2024`
	- Manually configure the time on the device.
- `calendar set CURRENT_TIME DAY MONTH YEAR`
	- E.g., `calendar set 14:58:00 4 Sep 2024`
	- Manually configure the hardware clock
Typically, 'clock' and 'calendar' should be synchronised.
- `clock update-calendar`
	- Update the calendar to match the clock's time.
- `clock read-calendar`
	- Update the clock to match the calendar's time.
### Time Zone Configuration
Time zone is configured in the global config mode, because it is a part of the device's running configuration.
- `clock timezone WORD HOURS MINUTES`
	- E.g., `clock timezone JST 9` (JST is 9 hours ahead of UTC, with no minutes offset)
	- `WORD`: Name of the time zone, device doesn't check if it's the name of a real time zone.
	- `HOURS`: Hours offset from UTC
	- `MINUTES`: Minutes offset from UTC
Daylight saving time - adjusting the times forward & backward once a year.
- `clock summer-time EDT recurring 2 Sunday March 02:00 1 Sunday November 02:00`
	- Terms used
		- `2 Sunday March 02:00` - Start of daylight saving time
		- `1` - hours to adjust
		- `1 Sunday November 02:00` - End of daylight saving time
	- Set the clock forward one hour on the second Sunday of March 2AM, then set the clock back one hour on the first Sunday of November at 2AM.
- Exact syntax from [Cisco documentation](https://www.cisco.com/c/en/us/td/docs/routers/xr12000/software/xr12k_r4-0/system_management/command/reference/yr40xr12k_chapter4.html): ![[Pasted image 20240904152633.png]]
## Network Time Protocol
A protocol that allows automatic syncing of time over a network. (Manually configuring the time on devices is not scalable - these will drift, causing in inaccurate time.)
- Cisco devices can operate in three NTP modes:
	- Server mode
	- Client mode
		- NTP clients request the time from NTP servers.
	- Symmetric active mode
		- Gets time from devices at the same stratum.
- A device can be an NTP server and an NTP client at the same time.
- NTP allows accuracy of time within:
	- ~1 millisecond if the NTP server is in the same LAN, or
	- ~50 milliseconds if connecting to the NTP server over a WAN/ the Internet.
- NTP uses UDP port 123 to communicate.
## NTP Hierarchy
- **Reference Clock**: A very accurate time device (atomic clock, GPS clock)
	- Original time sources
- **Stratum**: The '**distance**' of an NTP server from the original reference clock.
- The farther away from the reference clock, the higher the stratum. If the stratum level of a server is high, it is considered less accurate.
	- Reference clocks are **stratum 0** within the NTP hierarchy.
	- NTP servers directly connected to reference clocks are **stratum 1**.
	- Stratum 15 is the maximum, anything above that is considered unreliable.
![[Pasted image 20240904153959.png]]
- Stratum n+1 NTP servers get their time from stratum n NTP servers.
	- E.g., Primary servers (stratum 1 NTP servers) get their time from reference clocks, stratum 2 NTP servers get it from primary servers, etc. 
- Devices can 'peer' with devices at the same stratum to provide more accurate time.
	- A device is in a **symmetric active mode** if it obtains time from devices at the same stratum.
- An NTP client can sync to multiple NTP servers.
## NTP Configuration
![[Pasted image 20240904155349.png]]
![[Pasted image 20240904155545.png]]
- `nslookup NTP_DNS_ADDRESS`
	- A way to look up the IP address of an NTP server.
![[Pasted image 20240904155727.png]]
- `ntp server NTP_IP_ADDRESS [prefer]`
	- R1 will ask all addresses for the time and select the best, quickest responses.
	- Best to specify multiple addresses so R1 can get a reliable source of time.
	- Add `prefer` to make the address as a preferred address.
![[Pasted image 20240904155827.png]]
- `show ntp associations`
	- Shows all of the NTP servers that are configured.
	- `*` The NTP server R1 is currently syncing to. (This will constantly change as R1 continues interacting with the server.)
	- `+` Candidates, but R1 is not currently using it.
	- `~` Configured.
	- `-` Outlier, R1 will not sync to this server (It's inaccurate).
- `ref clock` shows the stratum 0 reference clock.
- `st` is 1 because the servers are connected directly to Google's own reference clock (it is a primary server).
![[Pasted image 20240904160202.png]]
- `show ntp status`
	- Stratum level is 2, because R1 is synchronizing its time to Google's NTP servers. (stratum level 1 higher than the primary servers)
	- R1 automatically becomes an NTP server itself. Now other devices can synchronize their time to R1.
- NTP uses only the UTC time zone, so you must configure the appropriate time zone on each device and update the calendar.

