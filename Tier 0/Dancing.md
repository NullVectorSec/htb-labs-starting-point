## 1. Questions
- Task 1
	- What does the 3-letter acronym SMB stand for?
		- `Server Message Block`
- Task 2
	- What port does SMB use to operate at?
		- `445`
- Task 3
	- What is the service name for port 445 that came up in our Nmap scan?
		- `microsoft-ds`
- Task 4
	- What is the 'flag' or 'switch' that we can use with the smbclient utility to 'list' the available shares on Dancing?
		- `-L`
- Task 5
	- How many shares are there on Dancing?
		- `4`
- Task 6
	- What is the name of the share we are able to access in the end with a blank password?
		- `WorkShares`
- Task 7
	- What is the command we can use within the SMB shell to download the files we find?
		- `get`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- smbclient
- dir
- more
## 3. Approach
### nmap scan
 - Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.1.12
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
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-13T00:28:00
|_  start_date: N/A
|_clock-skew: 3h59m58s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
<snip>
```
### Research common SMB usernames
- Resources
	- https://medium.com/@taliyabilal765/smb-enumeration-guide-b2cb5cfb20e6#:~:text=2%2D-,Common%20credentials,-%3A%20Try%20common
	- https://hackviser.com/tactics/pentesting/services/smb
- Example usernames
	- anonymous
	- admin
	- administrator
	- root
### Test SMB connection (passwordless)
- Useful resource
	- https://github.com/irgoncalves/smbclient_cheatsheet
- Successfully connected using username `anonymous`
```bash
smbclient -L //10.129.1.12 -U anonymous
```
- Result
```bash
Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	WorkShares      Disk      
<snip>
```
### Test share connection (passwordless)
- Useful resource
	- https://github.com/irgoncalves/smbclient_cheatsheet
- Successfully connected to the share
```bash
smbclient //10.129.1.12/WorkShares -U anonymous
```
### Explore files and directories
 - Useful resource
	- https://github.com/security-cheatsheet/cmd-command-cheat-sheet
- Flag found in `James.P` directory
```
flag.txt                            A       32  Mon Mar 29 04:26:57 2021
```
### Retrieve the flag
 Useful resource
	- https://github.com/security-cheatsheet/cmd-command-cheat-sheet
```bash
more ./James.P/flag.txt
```
