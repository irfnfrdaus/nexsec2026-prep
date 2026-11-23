# Web — Common CMS / App Weaknesses to Check

If the assigned web app is a known CMS/framework, check for the standard weak points first before
diving into custom code review.

## WordPress

```bash
# Version fingerprint
curl -s http://target/ | grep -i generator
cat wp-includes/version.php 2>/dev/null   # if you have filesystem access

# Enumerate plugins/themes (outdated = common entry point)
ls wp-content/plugins/ wp-content/themes/
# Check wp-config.php for exposed DB creds, debug mode left on
cat wp-config.php

# User enumeration
curl -s "http://target/?rest_route=/wp/v2/users"

# Check for known-bad: default/weak admin creds, XML-RPC enabled (brute force / amplification vector)
curl -s http://target/xmlrpc.php
```

## Generic PHP apps

- `config.php` / `.env` / `wp-config.php` left readable in the webroot with DB credentials.
- Default credentials unchanged on an admin panel.
- File upload fields with no extension/MIME validation (leads directly to webshells — cross-reference
  `01-webshell-detection.md`).
- `phpinfo()` or debug/test files left accessible (`info.php`, `test.php`, `phpmyadmin/` with default
  creds).

## Apache Tomcat / Java apps

```bash
curl -s http://target:8080/manager/html   # Tomcat Manager with default creds is a classic full-compromise path
```

## Outdated software fingerprinting

```bash
curl -sI http://target/     # Server header often reveals version
whatweb http://target/      # if available — fingerprints CMS, server, frameworks in one shot
nikto -h http://target/     # general web vuln scanner, flags outdated software + common misconfigs
```

## Directory/file exposure

```bash
gobuster dir -u http://target/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak,zip
# Look specifically for: .git/, .env, backup.zip, config.php.bak, admin/, uploads/
```

A `.git/` directory exposed on a live web server is a very common and very serious finding — it can
leak the entire source tree including past secrets:

```bash
curl -s http://target/.git/config
# If present, tools like git-dumper or wget -r can pull the whole repo for review
```

## Mapping findings to "the entry point"

For every weakness found, explicitly connect it to the incident: does it explain how the attacker got
in, or is it a separate, unrelated weakness you're fixing defensively? Both are worth reporting, but
the guideline specifically wants you to "understand the path an intrusion could take" — so prioritize
anything that plausibly explains the actual compromise you're investigating.

## Hardening quick wins

- [ ] Remove/restrict exposed `.git`, `.env`, backup files, debug endpoints.
- [ ] Disable directory listing (`Options -Indexes` in Apache, `autoindex off` in Nginx).
- [ ] Update the vulnerable component if a patch is available in the environment.
- [ ] Restrict file upload handling (extension whitelist, re-encode images, store outside the webroot
      or disable script execution in the upload directory).
- [ ] Change default/weak credentials on any admin panel found.
