## 1. Questions
- Task 1
	- Directory Brute-forcing is a technique used to check a lot of paths on a web server to find hidden pages. Which is another name for this? (i) Local File Inclusion, (ii) dir busting, (iii) hash cracking.
		- `dir busting`
- Task 2
	- What switch do we use for nmap's scan to specify that we want to perform version detection
		- `-sV`
- Task 3
	- What does Nmap report is the service identified as running on port 80/tcp?
		- `http`
- Task 4
	- What server name and version of service is running on port 80/tcp?
		- `nginx 1.14.2`
- Task 5
	- What switch do we use to specify to Gobuster we want to perform dir busting specifically?
		- `dir`
- Task 6
	- When using gobuster to dir bust, what switch do we add to make sure it finds PHP pages?
		- `-x php`
- Task 7
	- What page is found during our dir busting activities?
		- `admin.php`
- Task 8
	- What is the HTTP status code reported by Gobuster for the discovered page?
		- `200`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- curl
- gobuster
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.93.5
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
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome to nginx!
<snip>
```
### Test webserver connection
#### curl
-  Useful resource
	- https://devhints.io/curl
```bash
curl -i http://10.129.93.5
```
#### Browser
![](../Assets/Screenshot%202025-09-16%20at%2022.08.28.png)
### Enumerate directories
- Useful resource
	- https://hackertarget.com/gobuster-tutorial/#:~:text=in%20one%20location.-,Gobuster%20DIR%20command,-The%20DIR%20mode
```bash
gobuster dir -u http://10.129.93.5 -w /opt/useful/seclists/Discovery/Web-Content/common.txt
```
- Result
```bash
<snip>
/admin.php            (Status: 200) [Size: 999]
<snip>
```
### Explore /admin page
#### curl
-  Useful resource
	- https://devhints.io/curl
```bash
curl -i http://10.129.93.5/admin.php
```
#### Browser
![](../Assets/Screenshot%202025-09-16%20at%2022.07.17.png)
### Initial login form test
![](../Assets/Screenshot%202025-09-16%20at%2022.00.30.png)
### Research common admin credentials
- Resource
	- https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=Try%20the%20following%20usernames%20%2D%20%E2%80%9Cadmin,%E2%80%9Ctesting%E2%80%9D%20and%20similar%20names.
	- https://outpost24.com/blog/it-admins-weak-password-use/#:~:text=The%20top%2020%20administrator%20passwords
	- https://www.thehacker.recipes/web/config/default-credentials
	- https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=If%20a-,default%20password,-can%E2%80%99t%20be%20found
### Test admin login - using common credentials
- Test credentials
	- Usernames
		- `admin`
		- `administrator`
		- `root`
		- `system`
		- `guest`
		- `operator`
		- `super`
	- Passwords
		- `admin`
		- `Password`
		- `password`
		- `admin123`
		- `root`
- Successful login using username `admin` and password `admin`
![](../Assets/Screenshot%202025-09-16%20at%2022.06.02.png)
## 5. Tools Used
- Linux
- CLI
- nmap
- gobuster
