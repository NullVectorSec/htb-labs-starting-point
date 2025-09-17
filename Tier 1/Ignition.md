## 1. Questions
- Task 1
	- Which service version is found to be running on port 80?
		- `nginx 1.14.2`
- Task 2
	- What is the 3-digit HTTP status code returned when you visit `http://{machine IP}/`?
		- `302`
- Task 3
	- What is the virtual host name the webpage expects to be accessed by?
		- `ignition.htb`
- Task 4
	- What is the full path to the file on a Linux computer that holds a local list of domain name to IP address pairs?
		- `/etc/host`
- Task 5
	- Use a tool to brute force directories on the webserver. What is the full URL to the Magento login page?
		- `http://ignition.htb/admin`
- Task 6
	- Look up the password requirements for Magento and also try searching for the most common passwords of 2023. Which password provides access to the admin account?
		- `qwerty123`
- Submit Flag
	- Submit root flag
## 2. Tools
- Linux
- CLI
- nmap
- curl
- echo
- tee
- gobuster
## 3. Process
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.1.27
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
80/tcp open  http    nginx 1.14.2
|_http-title: Did not follow redirect to http://ignition.htb/
|_http-server-header: nginx/1.14.2
<snip>
```
### Webserver Response Testing
- Useful resource
	- https://devhints.io/curl
#### curl
```bash
curl -i 10.129.1.27
```
- Key Response Headers
	- `HTTP/1.1 302 Found`
	- `Location: http://ignition.htb/`
	- `X-XSS-Protection: 1; mode=block`
#### Redirect Testing
```bash
curl -i http://ignition.htb/
```
- Response:
	- `curl: (6) Could not resolve host: ignition.htb`
### Add Static Name Resolution Entry
- Based on redirect test response.
```bash
echo "10.129.1.27 ignition.htb" | sudo tee -a /etc/hosts
```
### Retest webserver response 
- Useful resource
	- https://devhints.io/curl
#### curl
```bash
curl -i http://ignition.htb/
```
#### Browser
![](../Assets/Screenshot%202025-08-20%20at%2020.21.51.png)
### Review webpage
- No elements of discernible value
### Directory enumeration
- Useful resource
	- https://3os.org/penetration-testing/cheatsheets/gobuster-cheatsheet/
```bash
gobuster dir -u http://ignition.htb -w /opt/useful/seclists/Discovery/Web-Content/common.txt
```
- Key directories found
	- `/0`
	- `/admin`
	- `/checkout/cart/`
	- `/cms`
	- `/contact`
	- `/enable-cookies`
	- `/errors/`
	- `/home`
	- `/index.php`
	- `/media/`
	- `/opt`
	- `/robots.txt`
	- `/robots`
	- `/setup`
	- `/soap`
	- `/static/`
	- `/customer/account/login/referer/aHR0cDovL2lnbml0aW9uLmh0Yi93aXNobGlzdA%2C%2C/`
### Testing directory 
- Most pages of no discernible value
#### /admin
![](../Assets/Screenshot%202025-08-20%20at%2020.49.09.png)
#### /customer/account/login/referer/aHR0cDovL2lnbml0aW9uLmh0Yi93aXNobGlzdA%2C%2C/
![](../Assets/Screenshot%202025-09-16%20at%2010.19.06.png)
### Research common credentials
- Usernames
	- https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=Try%20the%20following%20usernames%20%2D%20%E2%80%9Cadmin,%E2%80%9Ctesting%E2%80%9D%20and%20similar%20names.
- Passwords
	- https://nordpass.com/most-common-passwords-list/
### Form Response Testing
#### /admin
- Common credentials
	- Username
		- `admin`
		- `administrator`
		- `root`
		- `system`
		- `guest`
		- `operator`
		- `super`
	- Passwords
		- `123456`
		- `123456789`
		- `qwerty`
		- `password`
		- `12345`
		- `qwerty123`
		- `1q2w3e`
		- `12345678`
		- `111111`
		- `1234567890`
- Successfully logged in using username `admin` and password `qwerty123`
### Retrieve the flag
- Presented upon login
![](../Assets/Screenshot%202025-09-16%20at%2010.37.23.png)
