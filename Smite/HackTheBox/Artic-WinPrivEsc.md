# Artic - Its getting Cold...Fusion outside

<img src="https://media.tenor.com/0LhHcz68DMYAAAAM/cold-water.gif">

## Introduction
This machine was a part of the PNPT training course Windows Privilege Escalation - however it also highlights an initial vulnerability to get into the system

## Executive Summary

After initially scanning the machine with nmap and other tools, I was able to find a ColdFusion instance and admin login page. I found an existing metasploit module to exploit a vulnerability in the ColdFusion to get a low level shell on the system. I was then able to upgrade my shell to a meterpreter shell, which allowed me to enumerate the system further and find a kernel exploit to elevate my privileges to system, completely taking control of the machine.
