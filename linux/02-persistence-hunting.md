# Linux — Persistence Hunting

Once triage gives you a general picture, go mechanism-by-mechanism for how an attacker stays on the
box across reboots / logouts. Check **all** of these — real incidents (and this exercise) often stack
more than one.

## 1. SSH-based persistence

```bash
# Authorized keys for every user — the single most common persistence method
for home in /root /home/*; do
  f="$home/.ssh/authorized_keys"
  [ -f "$f" ] && echo "== $f ==" && cat "$f"
done

# SSH daemon config — check for backdoor settings
grep -En '^(PermitRootLogin|PasswordAuthentication|AuthorizedKeysFile|Port|PermitEmptyPasswords)' /etc/ssh/sshd_config

# Any secondary/custom sshd running on another port?
ps aux | grep sshd
ss -tlnp | grep ssh
```

Red flags: a key in `authorized_keys` with a comment you don't recognize, `PermitRootLogin yes`,
`AuthorizedKeysFile` pointed somewhere unusual (e.g. `/tmp/.keys`), a second sshd config/binary.

## 2. Cron & scheduled tasks

```bash
crontab -l -u root
for u in $(cut -f1 -d: /etc/passwd); do echo "== $u =="; crontab -l -u "$u" 2>/dev/null; done
cat /etc/crontab
ls -la /etc/cron.d/ /etc/cron.hourly/ /etc/cron.daily/ /etc/cron.weekly/ /etc/cron.monthly/
cat /etc/cron.d/* 2>/dev/null
systemctl list-timers --all
```

Red flags: a cron entry that curls/wgets and pipes to a shell, a job pointed at `/tmp` or `/dev/shm`,
a timer unit you don't recognize (`systemctl cat <name>.timer` to see what it triggers).

## 3. systemd services / units

```bash
systemctl list-units --type=service --all | grep -v 'loaded active'
systemctl list-unit-files --type=service | grep enabled
# Inspect anything suspicious:
systemctl cat <service-name>
ls -la /etc/systemd/system/ /usr/lib/systemd/system/ /lib/systemd/system/
```

Red flags: a service `ExecStart=` pointing at a script in `/tmp`, `/var/tmp`, a hidden dotfile
directory, or a one-liner reverse shell; a service that re-creates itself (watchdog pattern).

## 4. SUID/SGID binaries (privilege escalation persistence)

```bash
find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null -exec ls -la {} \;
```

Compare against a known-clean baseline if you have one (e.g. GTFOBins list of common legitimate SUID
binaries: `/usr/bin/passwd`, `/usr/bin/sudo`, `/usr/bin/su`, `/usr/bin/mount`, etc.). Anything outside
that set, or a *copy* of `bash`/`cp`/`python` with the SUID bit set, is a major red flag
(`cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash` is the classic pattern).

## 5. PAM / login backdoors

```bash
# PAM modules — a backdoored pam_unix.so or a rogue module in the auth stack is a serious, deep backdoor
ls -la /lib/x86_64-linux-gnu/security/ /lib/security/ 2>/dev/null
cat /etc/pam.d/sshd /etc/pam.d/common-auth 2>/dev/null
md5sum /lib/x86_64-linux-gnu/security/pam_unix.so 2>/dev/null   # compare against a known-good hash if available
```

## 6. Shell startup files & aliases

```bash
for f in /etc/profile /etc/bash.bashrc /root/.bashrc /root/.bash_profile /root/.profile; do
  echo "== $f =="; cat "$f" 2>/dev/null
done
# Also check every user's home directory
for home in /home/*; do
  cat "$home/.bashrc" "$home/.bash_profile" "$home/.profile" 2>/dev/null
done
```

Red flags: an alias that redefines `ls`/`sudo`/`su` to leak credentials; a startup file that curls and
executes something on every login.

## 7. LD_PRELOAD / shared library hijacking

```bash
cat /etc/ld.so.preload 2>/dev/null       # if this file exists and isn't empty, treat it as a major finding
env | grep LD_PRELOAD
find / -xdev -name "*.so" -newer /etc/hostname 2>/dev/null
```

## 8. Web shells (if the box also runs a web app — cross-reference with `web/01-webshell-detection.md`)

## 9. Kernel modules / rootkits (less likely in a time-boxed exercise, but check if time allows)

```bash
lsmod
# Compare against `modinfo` output for anything unrecognized; look for hidden modules via /proc/modules vs lsmod mismatches
chkrootkit 2>/dev/null || echo "chkrootkit not installed"
rkhunter --check 2>/dev/null || echo "rkhunter not installed"
```

## Remediation checklist once you've found something

- [ ] Remove the unauthorized SSH key / sudoers entry / cron job / systemd unit.
- [ ] Reset passwords for any account that was compromised or newly created.
- [ ] Kill the malicious process, then confirm it doesn't respawn (check what re-spawns it — a cron
      job, a systemd watchdog, a parent process).
- [ ] Patch the entry point if you identified one (outdated service, weak password, exposed port) —
      finding persistence without fixing the *cause* just means it comes back.
- [ ] Confirm required services are **still running** after your changes (the guideline explicitly
      grades this).
- [ ] Log everything you changed, in plain language, for the handover/report.
