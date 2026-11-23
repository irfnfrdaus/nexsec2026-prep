# Linux — Log Analysis

## Where the logs live

| Log | Path | What it tells you |
|---|---|---|
| Auth log | `/var/log/auth.log` (Debian/Ubuntu) or `/var/log/secure` (RHEL/CentOS) | SSH logins, sudo use, auth failures |
| System log | `/var/log/syslog` (Debian) or `/var/log/messages` (RHEL) | General system events |
| Bash history | `~/.bash_history` per user, `/root/.bash_history` | Commands run interactively (easily cleared/faked — treat with caution) |
| Login records | `/var/log/wtmp` (successful), `/var/log/btmp` (failed) | Binary logs, read with `last`/`lastb` |
| Package manager | `/var/log/apt/history.log` or `/var/log/yum.log` | Software installed/removed — can reveal attacker tooling |
| Cron log | `/var/log/cron` or in syslog | What cron jobs actually ran and when |
| Web server | `/var/log/apache2/access.log`, `/var/log/nginx/access.log` (+ error.log) | See `web/02-access-log-analysis.md` |
| Audit log (if auditd installed) | `/var/log/audit/audit.log` | Detailed syscall-level audit trail if configured |
| journald (systemd systems) | `journalctl` | Unified log — can replace several of the above |

## Fast commands

```bash
# Failed SSH logins — brute force indicator
grep "Failed password" /var/log/auth.log | tail -50
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head
# ^ counts failed attempts by source IP

# Successful logins
grep "Accepted" /var/log/auth.log

# sudo usage — who ran what as root
grep "sudo:" /var/log/auth.log

# New user / group creation
grep -E "new user|new group|useradd|groupadd" /var/log/auth.log /var/log/syslog

# journald equivalents
journalctl -u ssh --since "2 days ago"
journalctl _COMM=sudo --since today
journalctl --since "2026-11-25 09:00" --until "2026-11-25 12:00"   # tight time window around the incident
```

## Building a timeline

1. Get a reference point: when did the exercise/scenario start, or when was the system imaged/handed
   to you? Everything is analyzed relative to that.
2. Pull all auth events (logins, sudo, SSH) into chronological order.
3. Cross-reference with file modification times (`find / -newer <reference> -xdev`) and process start
   times (`ps -eo pid,lstart,cmd`).
4. Look for the **first anomalous event** — that's usually your initial access vector. Everything after
   it is attacker activity; everything before is baseline noise.

```bash
# Combine and sort multiple logs by timestamp for a quick manual timeline
cat /var/log/auth.log /var/log/syslog | sort -k1,3 -M | less
```

## Caveats

- `~/.bash_history` and logs can be cleared or tampered with by an attacker who got root — treat
  gaps/truncations as evidence in themselves ("log tampering" is a finding worth reporting).
- `HISTFILE` can be disabled (`unset HISTFILE`), so an empty bash history doesn't mean nothing happened
  — cross-check against auth logs, process accounting, and file timestamps.
- If `auditd` is installed and configured with rules, it's much harder to blind than bash history — check
  for it first (`systemctl status auditd`).

## What to put in your findings

For each anomalous log entry: **timestamp, source (IP/user/process), what happened, why it's
anomalous, and what evidence supports that conclusion.** This maps directly onto what the guideline
says will be reviewed — "findings a third party could verify."
