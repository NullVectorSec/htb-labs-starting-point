# htb-labs-starting-point
## Overview
This repo contains walkthroughs for the Hack the Box Starting Point machines. They are more focused on the steps taken and the outputs observed, opting for providing links to external resources for details on vulnerabilies, exploits, and tools used. 

The aim is to provide the reader with a more concise resource when working through the challenges, while still providing opportunities gain additional information with external links, should it be required.

## Machines
### Tier 0
| Machine Name                                                                                              | Services | Vulnerabilities/Exploits                   | Tools                       |
| --------------------------------------------------------------------------------------------------------- | -------- | ------------------------------------------ | --------------------------- |
| [Meow](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%200/Meow.md)               | telnet   | default credentials / lacks authentication | nmap, telnet                |
| [Fawn](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%200/Fawn.md)               | ftp      | anonymous authentication                   | nmap, ftp                   |
| [Dancing](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%200/Dancing.md)         | smb      | anonymous authentication                   | nmap, smbclient             |
| [Redeemer](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%200/Redeemer.md)       | redis    | unauthenticated access                     | nmap, redis-cli             |
| [Explosion](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%200/Explosion.md)     | smb, rdp | default credentials / lacks authentication | nmap, smbclient, xfreerdp   |
| [Preignition](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%200/Preignition.md) | http     | default credentials / weak authentication  | nmap, curl, gobuster        |
| [Mongod](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%200/Mongod.md)           | mongodb  | unauthenticated access                     | nmap, ssh, mongosh, mongodb |
| [Synced](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%200/Synced.md)           | rsync    | unauthenticated access                     | nmap, rsync                 |

### Tier 1
| Machine Name                                                                                              | Services      | Vulnerabilities/Exploits                                 | Tools                                        |
| --------------------------------------------------------------------------------------------------------- | ------------- | -------------------------------------------------------- | -------------------------------------------- |
| [Appointment](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Appointment.md) | http          | sql injection                                            | nmap, curl                                   |
| [Sequel](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Sequel.md)           | mysql         | default credentials / lacks authentication               | nmap, mysql                                  |
| [Crocodile](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Crocodile.md)     | ftp, http     | anonymous authentication / exposed credentials           | nmap, ftp                                    |
| [Responder](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Responder.md)     | http, WinRM   | parameter tampering, LFD, LFI, RFI                       | nmap, responder, john the ripper, evil-winrm |
| [Three](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Three.md)             | http, aws S3  | unrestricted file upload, RCE                            | nmap, gobuster, awscli, netcat               |
| [Ignition](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Ignition.md)       | http          | common credentials / weak authentication                 | nmap, curl, gobuster                         |
| [Bike](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Bike.md)               | http          | SSTI, RCE                                                | nmap, curl                                   |
| [Funnel](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Funnel.md)           | ftp ssh       | exposed credentials / default credentials / ssh pivoting | nmap, ftp, ssh, psql                         |
| [Pennyworth](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Pennyworth.md)   | http, jenkins | default credentials / weak authentication / RCE          | nmap, jenkins                                |
| [Tactics](https://github.com/NullVectorSec/htb-labs-starting-point/blob/main/Tier%201/Tactics.md)         | smb           | default credentials / lacks authentication               | nmap, smbclient, psexec.py                   |

## Tier 2
- In progress

## Important: General Disclaimer
- These walkthroughs are for educational purposes only.
- The techniques and tools shown are intended for use in lab environments or on systems where you have explicit permission.
- Under **no circumstances** should this content be used to compromise, attack, or gain unauthorised access to systems, networks, or data.
- All content represents the personal views and findings of the author and **does not represent** the views, policies, or positions of any employer, organisation, or platform used while creating the material.
- The author makes no warranties about the accuracy or fitness of the content
- The author accepts no liablity for any misuse or damage resulting from the use of the material.

## Important: External Links / Third-Party Content Disclaimer
- Any links provided are for reference only and the author does not control, endorse, or guarantee the accuracy, completeness, or safety of thrid-party content.
- External sites may change or remove content without notice and may contain content that is inappropriate, offensive, or harmful.
- Use caution when following external links: verify the source, check the content before running code, and do not download or execute files from untrusted sites.
- Use any and all external links solely at your own discretion and risk.
