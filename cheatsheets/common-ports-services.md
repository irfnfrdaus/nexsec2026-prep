# Common Ports & Services Quick Reference

Use during recon (`offensive/01-recon-enumeration.md`) to immediately know what to check once nmap
shows you an open port, without having to look it up mid-exercise.

| Port | Service | First things to check |
|---|---|---|
| 21 | FTP | Anonymous login (`anonymous`/blank or `anonymous`/`anonymous`), version for known CVEs |
| 22 | SSH | Version banner, weak/default creds, key-based auth misconfig |
| 23 | Telnet | Plaintext creds, default creds — presence alone is often a finding |
| 25 | SMTP | Open relay test, user enumeration via VRFY/EXPN |
| 53 | DNS | Zone transfer attempt (`dig axfr @target domain`), cache poisoning indicators |
| 80 / 443 | HTTP/HTTPS | Fingerprint stack (`whatweb`), directory brute-force, check for known CMS — see `web/` guides |
| 110 | POP3 | Plaintext creds in transit |
| 111 | RPCbind | `rpcinfo -p target` to enumerate registered RPC services (often leads to NFS) |
| 135 | MS-RPC (Windows) | Enumeration via `rpcclient`, often paired with 445 |
| 139 / 445 | SMB | Null session enumeration (`enum4linux`, `smbclient -L`), share permissions, EternalBlue-class CVEs on old versions |
| 143 | IMAP | Plaintext creds in transit |
| 389 / 636 | LDAP / LDAPS | Anonymous bind test, user/group enumeration — see `windows/05-active-directory-enumeration.md` |
| 443 | HTTPS | Same as 80 plus certificate info (`openssl s_client -connect target:443`) |
| 445 | SMB | See 139/445 above |
| 512-514 | r-services (rlogin/rsh/rexec) | Trust relationships, `.rhosts` misconfig — rare but a quick win if present |
| 1433 | MSSQL | Default `sa` account with blank/weak password, xp_cmdshell if accessible |
| 1521 | Oracle DB | Default accounts (`scott/tiger` etc.), TNS listener enumeration |
| 2049 | NFS | `showmount -e target`, check for `no_root_squash` exports — see `offensive/03-linux-privesc.md` |
| 3306 | MySQL | Default/weak root password, remote root login enabled |
| 3389 | RDP | Weak creds, check for BlueKeep-class CVEs on old versions, NLA enforcement |
| 5432 | PostgreSQL | Default `postgres` account, trust authentication misconfig |
| 5900 | VNC | Often no/weak authentication at all — try connecting directly |
| 5985 / 5986 | WinRM | Credential-based access if creds are found elsewhere (`evil-winrm`) |
| 6379 | Redis | Often no authentication by default — direct access can lead to RCE via config abuse |
| 8080 / 8000 / 8443 | Alt HTTP(S) / app servers | Same as 80/443; also check for admin panels (Tomcat Manager, Jenkins) |
| 9200 | Elasticsearch | Often no authentication by default — direct data access/RCE via scripting engines on old versions |
| 27017 | MongoDB | Often no authentication by default — direct data access |

## General triage logic once nmap gives you a port list

1. Anything with **no authentication by default** (Redis, MongoDB, Elasticsearch, sometimes VNC) →
   check immediately, these are often the fastest win.
2. Anything **database-related** → check default credentials before anything else.
3. Anything **Windows-specific** (135/139/445/389/3389) → cross-reference with
   `windows/` and `offensive/04-windows-privesc.md`.
4. **80/443/8080/etc.** → always worth a full pass regardless of what else is open, since web apps are
   the most common initial access point in these exercises.
