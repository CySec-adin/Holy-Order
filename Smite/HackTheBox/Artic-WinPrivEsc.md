# Artic - Its getting Cold...Fusion outside

<img src="https://media.tenor.com/0LhHcz68DMYAAAAM/cold-water.gif">

## Introduction
This machine was a part of the PNPT training course Windows Privilege Escalation - however it also highlights an initial vulnerability to get into the system

## Executive Summary

After initially scanning the machine with nmap and other tools, I was able to find a ColdFusion instance and admin login page. I found an existing metasploit module to exploit a vulnerability in the ColdFusion to get a low level shell on the system. I was then able to upgrade my shell to a meterpreter shell, which allowed me to enumerate the system further and find a kernel exploit to elevate my privileges to system, completely taking control of the machine.

## Initial Access

```
─$ sudo nmap -T4 -p- -A -oA Arctic 10.129.71.7 -Pn 
[sudo] password for kali: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-11-07 11:14 EST
Nmap scan report for 10.129.71.7
Host is up (0.047s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  http    JRun Web Server
49154/tcp open  msrpc   Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 8|Phone|2008|7|8.1|Vista|2012 (92%)
OS CPE: cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_8.1 cpe:/o:microsoft:windows_vista::- cpe:/o:microsoft:windows_vista::sp1 cpe:/o:microsoft:windows_server_2012
Aggressive OS guesses: Microsoft Windows 8.1 Update 1 (92%), Microsoft Windows Phone 7.5 or 8.0 (92%), Microsoft Windows 7 or Windows Server 2008 R2 (91%), Microsoft Windows Server 2008 R2 (91%), Microsoft Windows Server 2008 R2 or Windows 8.1 (91%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (91%), Microsoft Windows 7 (91%), Microsoft Windows 7 Professional or Windows 8 (91%), Microsoft Windows 7 SP1 or Windows Server 2008 R2 (91%), Microsoft Windows 7 SP1 or Windows Server 2008 SP2 or 2008 R2 SP1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 135/tcp)
HOP RTT      ADDRESS
1   50.14 ms 10.10.14.1
2   50.31 ms 10.129.71.7

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 232.18 seconds
```

Scanning with nmap revealed a few open ports, when visiting the web server running on port 8500 I was presented with an index page. Using gobuster I was able to find an admin login page for Cold Fusion, version 8

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/8abec8d0-8b9d-4a61-a492-a16118d48e63)

Searching through metasploit modules I was able to find an exploit to work for the version of Cold Fusion, and was also able to check for the directory needed to be attacked due to the index of the whole Cold Fusion directory being exposed. The exploit gained me a low level shell on the system (non-meterpreter). A meterpreter shell was first attempted with the exploit but was unattainable through the intial exploit. The shell gave me access to the account artic/tolis - a low level user - as shown below.

Metasploit module: windows/http/coldfusion_fckeditor

exposed directory:

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/e3e8293e-bb66-457e-8a64-2e04f676703f)

Exploited Vulnerability and Shell:

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/a8b4ea00-4638-4eee-a270-ac41b7b56443)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/69087693-0c23-43ca-987b-eb1c1a738e72)

## Privilege Escalation

Once in the system I was able to do a little enumeration of the system. I found the potential for a potato attack on the system due to privileges of the low level account, however I did not go this route and only explored it briefly.

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/5461d84d-9920-4e29-ab02-360e857d0712)

I then decided to try and upgrade my shell to a meterpreter shell, so that I could do more post exploitation enumeration of the system, and to test if there was an AV or some other security measure blocking the use of meterpreter. I was able to gain a meterpreter shell and run the post exploitation Local Exploit Suggester Module from metasploit (multi/recon/local_exploit_suggester).

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/4df75060-9723-4f3a-a7c3-67e5fbd06c32)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/4c690e93-267f-4880-a5be-2060081365f5)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/2b0edd6f-a62f-4809-a55e-e520c7eb02f8)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/50c8e588-081f-4c93-b652-2f49906d78ed)

With the Local Exploit Suggester I was able to find multiple potential kernel exploits to gain system level access, however I pursued the exploit ms10_092: https://learn.microsoft.com/en-us/security-updates/securitybulletins/2010/ms10-092 

Using this exploit, I had to migrate my meterpreter shell to a x64 process to execute the exploit, and was able to get system level access via the metasploit module (windows/local/ms10_092_schelevator) for this vulnerability. NOTE: the first screen capture is an error which let me know I needed to migrate to a different process first.

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/5c0eb2df-0ca0-41fe-950b-7844bc25b7cc)

Process Migration:

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/a7e6096d-b6fd-484b-93a0-b08fdc4f6a09)

Exploit:

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/96a111c3-8884-4db1-90b2-f437453ef583)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/c1761df7-8ef5-4244-95ea-2299cb66335d)

With this exploit successfully completed, I was able to gain full access to the machine.
