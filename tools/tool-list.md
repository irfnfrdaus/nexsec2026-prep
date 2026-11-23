# Tool List & Download Links

Install and get familiar with these **before** the event — remember, no AI assistance is allowed once
the exercise starts, including help with tool syntax, so practice now until the commands are muscle
memory.

## General / Linux IR & enumeration

| Tool | Purpose | Link |
|---|---|---|
| nmap | Port/service scanning | https://nmap.org/download.html |
| LinPEAS / WinPEAS (PEASS-ng) | Automated privesc enumeration | https://github.com/peass-ng/PEASS-ng |
| GTFOBins | Lookup table for sudo/SUID escape techniques | https://gtfobins.github.io/ |
| LOLBAS | Windows "living off the land" binary reference | https://lolbas-project.github.io/ |
| chkrootkit | Linux rootkit detection | http://www.chkrootkit.org/ |
| rkhunter | Linux rootkit detection | https://rkhunter.sourceforge.net/ |

## Web application testing

| Tool | Purpose | Link |
|---|---|---|
| Burp Suite (Community) | Web proxy / manual testing | https://portswigger.net/burp/communitydownload |
| gobuster | Directory/subdomain brute-forcing | https://github.com/OJ/gobuster |
| nikto | Web server vuln scanner | https://github.com/sullo/nikto |
| whatweb | Web fingerprinting | https://github.com/urbanadventurer/WhatWeb |
| sqlmap | SQL injection testing/exploitation | https://github.com/sqlmapproject/sqlmap |
| SecLists | Wordlists for everything above | https://github.com/danielmiessler/SecLists |

## Windows-specific

| Tool | Purpose | Link |
|---|---|---|
| Sysinternals Suite (Process Explorer, TCPView, Autoruns, etc.) | Live Windows triage | https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite |
| PowerSploit / PowerUp.ps1 | Privesc enumeration | https://github.com/PowerShellMafia/PowerSploit |
| Mimikatz | Credential extraction (authorized use only) | https://github.com/gentilkiwi/mimikatz |
| Impacket (secretsdump.py etc.) | Offline hash extraction, lateral movement tools | https://github.com/fortra/impacket |

## Password cracking

| Tool | Purpose | Link |
|---|---|---|
| hashcat | GPU-accelerated hash cracking | https://hashcat.net/hashcat/ |
| John the Ripper | CPU hash cracking | https://www.openwall.com/john/ |
| hydra | Online service brute-forcing | https://github.com/vanhauser-thc/thc-hydra |
| rockyou.txt | Standard password wordlist (ships with Kali at `/usr/share/wordlists/rockyou.txt.gz`) | — |

## Digital forensics (Day 2)

| Tool | Purpose | Link |
|---|---|---|
| Autopsy / The Sleuth Kit | Disk image analysis, timeline, file recovery | https://www.autopsy.com/download/ |
| Volatility 3 | Memory forensics | https://github.com/volatilityfoundation/volatility3 |
| Eric Zimmerman's Tools (MFTECmd, PECmd, RegRipper alt., Registry Explorer) | Windows artifact parsing | https://ericzimmerman.github.io/#!index.md |
| RegRipper | Registry hive parsing | https://github.com/keydet89/RegRipper3.0 |
| FTK Imager | Disk/memory imaging and preview | https://www.exterro.com/digital-forensics-software/ftk-imager |
| Plaso / log2timeline | Super-timeline generation | https://github.com/log2timeline/plaso |
| Wireshark | Packet capture analysis | https://www.wireshark.org/download.html |

## Environment to practice in beforehand

- **Hack The Box** (free tier covers many retired boot2root machines + a dedicated "Starting Point"
  track for beginners): https://www.hackthebox.com/
- **TryHackMe** (includes guided incident-response and forensics rooms, closer in spirit to this
  exercise's actual format than pure boot2root): https://tryhackme.com/
- **HTB Academy "Incident Handling Process" / "Digital Forensics" modules** — directly relevant given
  the exercise is IR + forensics, not pure exploitation: https://academy.hackthebox.com/

## A note on install timing

Download and test every tool above **before** the event. The organizer-provided laptops may have a
locked-down environment, so also check with the organizers (via the ticketing system, per the
guideline) whether you're expected to bring your own tooling or whether it's preloaded — don't assume.
