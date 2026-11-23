# Web — Access Log Analysis

## Log locations

| Server | Access log | Error log |
|---|---|---|
| Apache | `/var/log/apache2/access.log` (Debian) / `/var/log/httpd/access_log` (RHEL) | `error.log` in same dir |
| Nginx | `/var/log/nginx/access.log` | `/var/log/nginx/error.log` |
| IIS | `C:\inetpub\logs\LogFiles\W3SVC1\` (W3C format) | Windows Event Log (Application) |

## Fast triage queries (Apache/Nginx combined log format)

```bash
# Top requested URLs — spot a webshell or admin panel getting hammered
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -30

# Top client IPs by request count
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# Requests with suspicious status codes clustered from one IP (scanning behavior)
awk '$9 ~ /^4|^5/ {print $1}' access.log | sort | uniq -c | sort -rn | head -20

# POST requests to unusual-looking files (upload endpoints, odd extensions)
grep '"POST' access.log | awk '{print $7}' | sort | uniq -c | sort -rn

# Common attack signatures in the request line
grep -Ei "union(.*)select|<script>|\.\./\.\./|/etc/passwd|cmd=|eval\(|base64_decode" access.log

# SQLi-looking query strings
grep -Ei "(\%27)|(\')|(--)|(\%23)|(#)" access.log | grep -i "select\|union\|insert\|drop"

# Requests to the webshell file you found in 01-webshell-detection.md
grep "shell_filename.php" access.log
```

## Building a request timeline for one attacker/session

```bash
grep "<attacker-ip>" access.log | sort -k4 -t'[' | less
# [ splits the Apache common-log timestamp field — sort chronologically on it
```

Walk through chronologically: look for the pattern **recon → exploitation attempt → successful
exploitation (often a status 200 on something unusual) → webshell access → further requests from
the shell (often POST requests, or GET with command parameters).**

## User-Agent anomalies

```bash
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -rn | head -20
```

Automated tools (sqlmap, nikto, dirb/gobuster, curl with no UA override) often leave a distinctive
or missing User-Agent — useful for separating recon noise from the real exploitation request.

## IIS / W3C log notes

IIS logs are space-delimited with a header line defining columns (`#Fields:`). Use that header to find
the right column instead of guessing:

```powershell
Get-Content u_ex*.log | Select-String "^#Fields" | Select -First 1
Import-Csv -Delimiter ' ' -Header (the field list from above) u_ex*.log |
  Group-Object cs-uri-stem | Sort Count -Descending | Select -First 20
```

## What to extract for your report

- First request that indicates successful exploitation (with timestamp, source IP, exact request).
- Every subsequent request that appears to interact with the planted webshell/backdoor.
- Any evidence of data exfiltration (large response sizes, requests to export/download endpoints).
- Source IP(s) involved — note whether they match the scenario's "attacker" indicators if the
  organizers gave you any (e.g. a provided IOC list).
