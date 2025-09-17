## 1. Questions
- Task 1
	- How many TCP ports are open on the machine?
		- `2`
- Task 2
	- Which service is running on port 27017 of the remote host?
		- `MongoDB 3.6.8`
- Task 3
	- What type of database is MongoDB? (Choose: SQL or NoSQL)
		- `NoSQL`
- Task 4
	- What command is used to launch the interactive MongoDB shell from the terminal?
		- `mongosh`
- Task 5
	- What is the command used for listing all the databases present on the MongoDB server? (No need to include a trailing ;)
		- `show dbs`
- Task 6
	- What is the command used for listing out the collections in a database? (No need to include a trailing ;)
		- `show collections`
- Task 7
	- What command is used to dump the content of all the documents within the collection named `flag`?
		- `db.flag.find()`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- ls
- grep
- mongosh
- curl
- tar
- mongodb
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -p- -T4 10.129.99.146
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
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
27017/tcp open  mongodb MongoDB 3.6.8 3.6.8
| mongodb-databases: 
|   databases
|     0
|       sizeOnDisk = 32768.0
|       empty = false
|       name = admin
|     2
|       sizeOnDisk = 73728.0
|       empty = false
|       name = local
|     1
|       sizeOnDisk = 73728.0
|       empty = false
|       name = config
|     4
|       sizeOnDisk = 32768.0
|       empty = false
|       name = users
|     3
|       sizeOnDisk = 32768.0
|       empty = false
|       name = sensitive_information
|   ok = 1.0
|_  totalSize = 245760.0
<snip>
```
### Research common credentials
- Usernames
	- https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials
- Passwords
	- https://outpost24.com/blog/it-admins-weak-password-use/#:~:text=The%20top%2020%20administrator%20passwords
### Test ssh connection
- Useful resource
	- https://quickref.me/ssh.html
- Test usernames
	- `anonymous`
	- `admin`
	- `administrator`
	- `root`
	- `test`
- Test passwords
	- admin
	- password
	- admin123
	- root
	- admin@123
	- admin1234
- Unable to connect using test credentials
### Check mongosh installed
- Useful resource
	- https://www.mongodb.com/docs/mongodb-shell/install/#confirm-that-mongosh-installed-successfully
- `v2.3.8` installed
```bash
mongosh --version
```
### Test connection to mongodb
- Useful resource
	- https://www.mongodb.com/docs/v7.0/reference/cheatsheet/#connect
```bash
mongosh mongodb://10.129.99.146:27017
```
- Resulted in dependency version conflict error
```bash
MongoServerSelectionError: Server at 10.129.99.146:27017 reports maximum wire version 6, but this version of the Node.js Driver requires at least 7 (MongoDB 4.0)
```
### Test previous versions of mongosh
- Useful resource
	- https://www.mongodb.com/docs/mongodb-shell/changelog/#v2.3.6
	- https://www.mongodb.com/docs/mongodb-shell/changelog/#v2.3.2
	- https://www.mongodb.com/docs/v7.0/reference/cheatsheet/#connect
- Focus
	- Previous versions with no `MONGOSH-*` marked changes
- Versions tested
	- `v2.3.6`
	- `v2.3.2`
- Result
	- `v2.3.2` connects successfully without credentials
```bash
./mongosh-2.3.2-linux-x64/bin/mongosh mongodb://10.129.99.146:27017
```
### Explore databases
- Useful resource
	- https://www.mongodb.com/docs/v7.0/reference/cheatsheet/#show-databases
```mongodb
show dbs
```
- Result
```mongodb
admin                  32.00 KiB
config                 72.00 KiB
local                  72.00 KiB
sensitive_information  32.00 KiB
users                  32.00 KiB
```
### Switch to sensitive_information database
- Useful resource
	- https://www.mongodb.com/docs/v7.0/reference/cheatsheet/#switch-database
```sql
use sensitive_information
```
### Explore collections
- Useful resource
	- https://www.mongodb.com/docs/v7.0/reference/cheatsheet/#show-collections
```sql
show collections
```
- Result
```sql
flag
```
### Retrieve the flag
- Useful resource
	- https://www.mongodb.com/docs/v7.0/reference/cheatsheet/#read
```bash
db.flag.find().pretty()
```
## 4. Installing v2.3.2 mongosh
### Download the archive
```bash
curl -O https://downloads.mongodb.com/compass/mongosh-2.3.2-linux-x64.tgz;
```
### Decompress the archive
```basch
tar xvf mongosh-2.3.2-linux-x64.tgz;
```
