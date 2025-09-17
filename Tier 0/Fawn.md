## 1. Questions
- Task 1
	- What does the 3-letter acronym FTP stand for?
		- `File Transfer Protocol`
- Task 2
	- Which port does the FTP service listen on usually?
		- `21`
- Task 3
	- FTP sends data in the clear, without any encryption. What acronym is used for a later protocol designed to provide similar functionality to FTP but securely, as an extension of the SSH protocol?
		- `SFTP`
- Task 4
	- What is the command we can use to send an ICMP echo request to test our connection to the target?
		- `ping`
- Task 5
	- From your scans, what version is FTP running on the target?
		- `vsftpd 3.0.3```
- Task 6
	- From your scans, what OS type is running on the target?
		- `Unix`
- Task 7
	- What is the command we need to run in order to display the 'ftp' client help menu?
		- `ftp -?`
- Task 8
	- What is username that is used over FTP when you want to log in without having an account?
		- `anonymous`
- Task 9
	- What is the response code we get for the FTP message 'Login successful'?
		- `230`
- Task 10
	- There are a couple of commands we can use to list the files and directories available on the FTP server. One is dir. What is the other that is a common way to list files on a Linux system.
		- `ls`
- Task 11
	- What is the command used to download the file we found on the FTP server?
		- `get`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- ftp
- lftp
- tnftp
- ls
- less
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.96.171
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
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.85
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
Service Info: OS: Unix
<snip>
```
### Research common usernames
- https://hackviser.com/tactics/pentesting/services/ftp#anonymous-authentication
- https://hackviser.com/tactics/pentesting/services/ftp#common-credentials
### Test ftp connection (passwordless)
- Useful resource
	- https://cheatsheet.haax.fr/network/services-enumeration/21_ftp/
	- https://lftp.yar.ru/lftp-man.html
	- https://www.mankier.com/1/tnftp
- Test usernames
	- `anonymous`
	- `admin`
	- `administrator`
	- `root`
	- `ftpuser`
	- `test`
- Successfully connected using username `anonymous`
```bash
ftp anonymous@10.129.96.171
lftp anonymous@10.129.96.171
tnftp anonymous@10.129.96.171
```
### Explore files and directories
- Flag found in login directory
```bash
<snip>
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
<snip>
```
### Retrieve the flag
```bash
less flag.txt
```
