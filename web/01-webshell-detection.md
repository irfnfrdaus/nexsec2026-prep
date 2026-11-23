# Web — Webshell & Backdoor Detection

If an assigned host runs a web application, assume a webshell or backdoored code is a strong
possibility — it's the most common way attackers maintain access to a compromised web server.

## 1. Find recently modified files in the webroot

```bash
# Linux — common webroots: /var/www/html, /var/www, /srv/http
find /var/www -type f \( -name "*.php" -o -name "*.jsp" -o -name "*.asp*" \) -newer /etc/hostname 2>/dev/null
find /var/www -type f -mtime -3   # modified in the last 3 days

# Windows — common webroots: C:\inetpub\wwwroot, C:\xampp\htdocs
Get-ChildItem C:\inetpub\wwwroot -Recurse -Include *.aspx,*.asp,*.php |
  Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-3) }
```

## 2. Grep for common webshell signatures

```bash
# PHP
grep -rliE "eval\s*\(|base64_decode\s*\(|system\s*\(|shell_exec\s*\(|passthru\s*\(|assert\s*\(|exec\s*\(" /var/www --include="*.php"
grep -rliE "\\\$_(GET|POST|REQUEST)\[.{1,30}\]\s*\(" /var/www --include="*.php"

# Common known webshell names worth searching for directly (attackers often reuse public shells)
find /var/www -iname "*shell*" -o -iname "*c99*" -o -iname "*r57*" -o -iname "*wso*" -o -iname "*b374k*"
```

```powershell
# ASP/ASPX equivalent grep
Get-ChildItem C:\inetpub\wwwroot -Recurse -Include *.asp,*.aspx |
  Select-String -Pattern "eval|Execute|CreateObject\(.Wscript.Shell.\)|Request\(" |
  Select Path, LineNumber, Line
```

## 3. Look for files that don't match the application's expected structure

- A single `.php`/`.jsp`/`.aspx` file sitting in an upload directory (`/uploads/`, `/images/`,
  `/tmp/`) that normally only holds media — classic sign of an unrestricted file upload exploit.
- Obfuscated code: long base64 blobs, heavy use of `chr()`/string concatenation to hide function names,
  minified single-line files with no formatting.
- Files with suspicious permissions (world-writable, or owned by the web server user when the rest of
  the app is owned by a deploy user).

```bash
find /var/www -perm -0002 -type f 2>/dev/null
```

## 4. Check web server access logs for the shell being accessed

Once you find a suspicious file, grep the access log for requests to it — this tells you when it was
planted/used and from where (see `02-access-log-analysis.md` for the full workflow).

```bash
grep "suspicious-file.php" /var/log/apache2/access.log /var/log/nginx/access.log
```

## 5. Check for malicious .htaccess / web.config entries

```bash
find /var/www -name ".htaccess" -exec cat {} \;
```

Red flags: rewrite rules that redirect based on `User-Agent` or `Referer` (cloaking), rules that
silently proxy requests, PHP execution enabled in directories that shouldn't allow it.

## Remediation

- [ ] Remove the webshell file (copy aside for evidence first).
- [ ] Identify and patch the vulnerability that allowed it to be planted (unrestricted upload, LFI/RFI,
      outdated CMS plugin, exposed admin panel with default creds, etc.) — removing the shell without
      fixing the entry point means it comes right back.
- [ ] Review all other files modified around the same time window — a single shell is rarely the only
      change.
- [ ] Rotate any credentials the web app had access to (DB creds, API keys) if there's a chance they
      were read by the shell.
