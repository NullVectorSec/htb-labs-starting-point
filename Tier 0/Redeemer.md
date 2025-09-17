## 1. Questions
- Task 1
	- Which TCP port is open on the machine?
		- `6379`
- Task 2
	- Which service is running on the port that is open on the machine?
		- `redis`
- Task 3
	- What type of database is Redis? Choose from the following options: (i) In-memory Database, (ii) Traditional Database
		- `In-memory Database`
- Task 4
	- Which command-line utility is used to interact with the Redis server? Enter the program name you would enter into the terminal without any arguments.
		- `redis-cli`
- Task 5
	- Which flag is used with the Redis command-line utility to specify the hostname?
		- `-h`
- Task 6
	- Once connected to a Redis server, which command is used to obtain the information and statistics about the Redis server?
		- `info`
- Task 7
	- What is the version of the Redis server being used on the target machine?
		- `5.0.7`
- Task 8
	- Which command is used to select the desired database in Redis?
		- `select`
- Task 9
	- How many keys are present inside the database with index 0?
		- `4`
- Task 10
	- Which command is used to obtain all the keys in a database?
		- `keys *`
- Submit Flag
	- Submit root flag
## 2. Tools Used
- Linux
- CLI
- nmap
- redis-cli
## 3. Approach
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -p- -T4 10.129.181.163
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
PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7
```
### Research common redis credentials
- https://hackviser.com/tactics/pentesting/services/redis#passwordless-authentication
- https://hackviser.com/tactics/pentesting/services/redis#common-credentials
### Test redis connection
- Useful resource
	- https://hackviser.com/tactics/pentesting/services/redis#passwordless-authentication
- Successfully connected without credentials
```bash
redis-cli -h 10.129.181.163
```
### Explore keys
- Useful resource
	- https://medium.com/@shaskumar/redis-scan-vs-keys-command-9df7f51b7162#:~:text=your%20use%20case.-,KEYS,-This%20is%20a
```bash
keys *
```
- Result
```
1) "numb"
2) "temp"
3) "stor"
4) "flag"
```
### Retrieve the flag
- Useful resource
	- https://redis.io/docs/latest/commands/get/
```bash
get flag
```
