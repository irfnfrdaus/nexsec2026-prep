# Linux — Initial Triage

Goal: in the first 10–15 minutes on an assigned Linux box, build a picture of what's running, who's
logged in, and what looks wrong — before you start hunting for specific persistence mechanisms.

## 1. Identity & environment

```bash
hostname; uname -a; cat /etc/os-release
whoami; id
uptime
```

## 2. Who's on the box right now / recently

```bash
w                      # logged in users + what they're doing
who -a
last -a | head -30     # login history (reads wtmp)
lastb | head -30        # failed logins (reads btmp) — needs root
lastlog                # last login per user — flags accounts that have "never" logged in but exist
```

Red flags: logins from unfamiliar IPs, service accounts with interactive shells logging in, logins at
odd hours relative to the scenario timeline.

## 3. User accounts

```bash
cat /etc/passwd            # look for unexpected UID 0 accounts, or accounts with /bin/bash that shouldn't have one
awk -F: '$3 == 0 {print}' /etc/passwd     # all UID 0 (root-equivalent) accounts — should normally be just root
cat /etc/shadow 2>/dev/null | awk -F: '{print $1,$2}'   # account lock status (! or * = locked)
cat /etc/sudoers; ls /etc/sudoers.d/      # who has sudo, and how
grep -v -E '^\s*#|^\s*$' /etc/sudoers /etc/sudoers.d/* 2>/dev/null
```

Red flags: new user with UID 0, a low-privilege-looking account with blanket `ALL=(ALL) NOPASSWD:ALL`,
accounts not in `/etc/passwd` on a fleet baseline you were given.

## 4. Running processes & network

```bash
ps auxww --forest           # full process tree
ps -eo pid,ppid,user,cmd --sort=-%cpu | head -20
ss -tulpn                   # listening ports + owning process (use `netstat -tulpn` if ss missing)
ss -antp | grep ESTAB       # established connections — look for outbound to odd IPs/ports
lsof -i 2>/dev/null         # alternative view of network-bound processes
```

Red flags: processes running from `/tmp`, `/dev/shm`, `/var/tmp`; processes with deleted binaries
(`ls -la /proc/<pid>/exe` shows `(deleted)`); unexpected listeners on high ports; reverse shells
(outbound connections from odd binaries like `bash`, `python`, `nc` to non-standard ports).

```bash
# Find processes whose binary no longer exists on disk (common malware trick)
for p in /proc/[0-9]*; do
  pid=$(basename "$p")
  exe=$(readlink "$p/exe" 2>/dev/null)
  [[ "$exe" == *"(deleted)"* ]] && echo "PID $pid -> $exe"
done
```

## 5. Quick filesystem sanity check

```bash
df -h                       # disk usage — full /tmp or /var can indicate dropped payloads/logs
find / -xdev -newer /etc/hostname -type f 2>/dev/null | grep -vE '^/proc|^/sys' | head -50
# ^ files modified since a known-old reference point — tune the reference file to something
#   you know predates the incident (or use `find / -mtime -2` for "modified in last 2 days")
```

## 6. What's scheduled to run

Covered in depth in `02-persistence-hunting.md`, but a first pass:

```bash
crontab -l -u root; for u in $(cut -f1 -d: /etc/passwd); do crontab -l -u "$u" 2>/dev/null; done
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/ /etc/cron.weekly/ /etc/cron.monthly/
cat /etc/crontab
systemctl list-timers --all
```

## Triage priority order

1. Who can log in / escalate (accounts, sudo, SSH keys) — because this is how the attacker *stays* in.
2. What's listening / phoning home (network) — active C2 or backdoor.
3. What's scheduled to run (cron/systemd timers) — persistence that survives reboot.
4. What's changed on disk (timestamps, unexpected files) — scope the blast radius.

Write down what you find as you go — don't rely on memory. A simple running notes file
(`notes.md` on your own analysis machine, not the target) saves you when the team needs a handover.
