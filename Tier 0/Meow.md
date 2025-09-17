## 1. Questions
- Task 1
	- What does the acronym VM stand for?
		- `Virtual Machine`
- Task 2
	- What tool do we use to interact with the operating system in order to issue commands via the command line, such as the one to start our VPN connection? It's also known as a console or shell.
		- `terminal`
- Task 3
	- What service do we use to form our VPN connection into HTB labs?
		- `openvpn`
- Task 4
	- What tool do we use to test our connection to the target with an ICMP echo request?
		- `ping`
- Task 5
	- What is the name of the most common tool for finding open ports on a target?
		- `nmap`
- Task 6
	- What service do we identify on port 23/tcp during our scans?
		- `telnet`
- Task 7
	- What username is able to log into the target over telnet with a blank password?
		- `anonymous`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- telnet
- ls
- cat
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.1.17
```
- Flags
	- `-sSVC` - SYN, service, and default sript scan.
	- `-Pn` - Exclude ICMP scanning.
	- `-n` - Exclude name resolution.
	- `--disable-arp-ping` - Exclude `arp` scanning.
	- `-T4` - Use speed template `aggressive`.
- Result
```bash
<snip>
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
<snip>
```
### Research common usernames
- https://hackviser.com/tactics/pentesting/services/telnet#passwordless-authentication
- https://hackviser.com/tactics/pentesting/services/telnet#common-credentials
### Test telnet connection (passwordless)
- Useful resource
	- https://cheat.sh/telnet
- Test usernames
	- `anonymous`
	- `admin`
	- `administrator`
	- `root`
	- `test`
- Successfully connected using username `root`
```bash
telnet -l root 10.129.1.17 23
```
### Explore files and directories
- Flag found in login directory
```bash
<snip>
-rw-r--r--  1 root root   33 Jun 17  2021 flag.txt
<snip>
```
### Retrieve the flag
```bash
cat flag.txt
```
