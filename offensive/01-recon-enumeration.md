# Offensive — Recon & Enumeration (General CTF / HTB-style)

Included in case the actual exercise format leans more offensive than the official guideline suggests.
**Remember:** in the real exercise, only scan/attack systems explicitly assigned to your team, only
through the provided access path (Section 6.1 of the rules).

## Network/host discovery

```bash
nmap -sn 10.0.0.0/24                       # host discovery (ping sweep)
nmap -p- -T4 -oA full_tcp <target>          # full TCP port sweep, all 65535 ports
nmap -sV -sC -p<ports> -oA detailed <target>  # service/version detection + default scripts on found ports
nmap -sU --top-ports 20 <target>            # quick UDP check (full UDP scan is slow — prioritize)
```

Work from `full_tcp` results → re-scan just the open ports with `-sV -sC` for detail. Don't run `-A`/
heavy scans blind against a full /24 — it's slow and noisy.

## Web enumeration

```bash
whatweb <target>                            # fingerprint CMS/server/framework
gobuster dir -u http://<target>/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,html,bak
gobuster dns -d <target-domain> -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
nikto -h http://<target>/                   # general vuln/misconfig scan
```

Always check manually too: view source, robots.txt, sitemap.xml, common admin paths
(`/admin`, `/login`, `/wp-admin`, `/manager/html`), and HTTP response headers for version info.

## SMB / Windows service enumeration

```bash
nmap -p139,445 --script smb-enum-shares,smb-enum-users,smb-os-discovery <target>
smbclient -L //<target>/ -N                 # list shares, null session
enum4linux -a <target>                      # broader SMB/AD enumeration
crackmapexec smb <target> -u '' -p ''       # null session check across a range
```

## SNMP / other common services

```bash
snmpwalk -c public -v1 <target>             # default community string check
onesixtyone -c community-strings.txt <target>
```

## FTP / anonymous access

```bash
ftp <target>                                 # try anonymous:anonymous
nmap -p21 --script ftp-anon <target>
```

## Building an attack surface map

Keep a running list per target: IP, open ports, service+version per port, any credentials found, any
notes on what looked interesting. This is the same discipline as the IR note-taking in the other
guides — a messy recon phase costs far more time later than a structured one.

## General principle

Enumeration is almost always the bottleneck, not exploitation. A thorough pass here (every open port
checked, every web path crawled, every service version noted) pays off more than rushing to try
exploits against a half-understood target.
